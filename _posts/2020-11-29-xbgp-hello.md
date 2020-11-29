---
title:  "A first xBGP plugin"
date:   2020-11-29
categories: xbgp
author: thomas
---


An xBGP compliant implementation such as our forks of [BIRD](https://github.com/pluginized-protocols/xbgp_bird) and [FRRouting](https://github.com/pluginized-protocols/xbgp_frr) supports a simple API that enables network operators and researchers to extend it. The xBGP API is defined and briefly documented in [xbgp_plugin_api.h](https://github.com/pluginized-protocols/xbgp_plugins/blob/master/xbgp_compliant_api/xbgp_plugin_api.h). In this post, we start with a very simple xBGP extension that we implement using a plugin.

The BGP specifications define the format of the BGP messages. The UPDATE message is the most complex BGP message since it contains the information about the new routes. BGP supports different BGP Path Attributes that are defined in various RFCs. Each of these path attributes is identified using a one byte integer. IANA maintains a [list with all the known path attributes](https://www.iana.org/assignments/bgp-parameters/bgp-parameters.xhtml#bgp-parameters-2).

For our first example, we simply assume that a network operator would like to ignore the BGP UPDATE messages that contain an unknown attribute. A practical example of this usage is when problems with the processing of [BGP Path attribute 128](https://kb.juniper.net/InfoCenter/index?page=content&id=JSA10491) caused the failure of BGP sessions. To support this feature, we retrieve the list of allocated BGP path attribute identifiers from IANA. We use it to create the `is_known_attr` macro that checks this validity.

```c
#include "ubpf_api.h"
#include "bytecode_public.h"

#define is_known_attr(code) ( \
    ((code) == RESERVED_ATTR_ID) || \
    ((code) == ORIGIN_ATTR_ID) || \
    ((code) == AS_PATH_ATTR_ID) || \
    ...
    )
```

This filter needs to be attached to the xBGP implementation at the code point where it parses an incoming BGP message, i.e., at location `1` in the figure below.

![BGP insertion points]({{ site.baseurl }}/images/bgp-points.png)

The next step is to write a small C function that parses the received message and checks the attributes that it contains.

```c
uint64_t parse_attribute(args_t *args UNUSED) {

    uint8_t *code;
    code = get_arg(0); // argument 0 is the code attribute received from the neighbor.

    if (!code) {
        // unable to retrieve the argument (internal failure)
        return EXIT_FAILURE;
    }

    if (!is_known_attr(*code)) {
        return EXIT_FAILURE;
    }

    return 0;
}
```

You can experiment with this simple plugin as follows. Save your plugin as `valid_attr.c`.

First, the C program must be compiled to an eBPF bytecode. This can be done with the clang compiler.
However for the sake of simplicity, we provide a Makefile that contains the prebuilt commands to
successfully compile your plugins.

Clone our GitHub repository:

```bash
$ git clone https://github.com/pluginized-protocols/xbgp_plugins.git
```

The libxbgp folder must also be cloned. It contains headers that are required to interact with the eBPF userspace virtual
machine. For example, they contain functions to allocate memory, retrieve arguments (`get_arg`) from the host implementation and using
base functions such as `htonl`, `ntohl`, etc.

All your plugins must be located in the cloned folder because it contains the required headers that define the xBGP API used by the plugins.

Inside the `xbgp_plugins` folder, execute the following command:

```bash
$ make LIBXBGP=<path/to/the/libxbgp cloned repository>
```

This command will browse the entire folder and its subfolders recursively to find any `.c` files. For each `.c` file, the clang compiler will produce the associated object (`.o`) file. This will be the eBPF bytecode of your plugin.

Take the `valid_attr.o` associated with the function written previously. You are now ready to inject it inside one xBGP compliant
BGP implementation. In this tutorial, we provide the steps to successfully update the FRRouting BGPD daemon. The steps are similar for BIRD routing.

Our modified version of FRR BGPD requires that each `.o` must be located in `<config_frr_dir>/frr/plugins`. By default, and
depending on the compilation of FRRouting, the default configuration folder is  `/etc/frr/plugins`.

The next step is to update the manifest. This manifest is a json formatted file that is read by the daemon
at startup to include all the plugins that it contains. It is located at
`<config_frr_dir>/frr/plugins/manifest.json`. Modify it as follows: 

```json
{
  "jit_all": false,
  "plugins": {
    "filter_invalid_attr": {
      "extra_mem": 128,
      "shared_mem": 0,
      "obj_code_list": {
        "invalid_attr": {
          "obj": "invalid_attr.o",
          "jit": true
        }
      }
    }
  },
  "insertion_points": {
    "bgp_decode_attr": {
      "replace": {
        "0": "invalid_attr"
      }
    }
  }
}
```

This manifest contains several required fields :

* `plugins`: contains the definition of all that plugins that must be loaded at startup time.
    * `<plugin_name>`: the identifier of the plugin. In this example, we use `filter_invalid_attr`
        * `extra_mem`: how many _*bytes*_ to allocate for the API function. Some functions need extra memory to allocate
                       data retrieved from the host implementation. Other functions such as `ctx_malloc` rely on this
                       memory space.
        * `shared_mem`: how many _*bytes*_ to allocate for the memory space shared among the pluglets of the same plugin.
                        This memory space is used by functions of type `shared_memory`.
        * `obj_code_list`: contains the list of the pluglets that belong to the same plugin. Each pluglet must have a unique
                           identifier. In the example, we use the identifier `invalid_attr`.
            * `<pluglet_identifier>`: the identifier associated with the actual eBPF bytecode object.
                * `obj`: the real name of the file containing the eBPF bytecode.
                * `jit`: whether or not the pluglet must be converted to `x86_64` machine code (can be omitted. Default: false).
* `insertion_points`: this object field defines the location of each pluglet of a given plugin inside the host BGP
                      implementation. This field takes a list of insertion point identifiers. We want that our
                      plugin composed of one pluglet is executed at  `bgp_decode_attr`.
    * `<insertion_point_id>`: defines the list of pluglets that must be executed at this insertion_point
        * `<replace|pre|post>`: contains all the pluglets that will be executed in the `replace`, `pre` or `post`
                                part of the insertion_point.
            * `<execution order (int)>`: the value associated to the int is the pluglet identifier defined in the
                                         plugin part of the manifest. In our example we want to execute the `invalid_attr.o`
                                         bytecode as the first pluglet of this insertion point. In the
                                         case of multiple insertion point the `execution order` field determines the order
                                         of execution. Pluglet `0` is executed first, then `1`, ...


Once the manifest has been saved and the pluglets saved in the `<config_frr_dir>/etc/plugins` file, you are now ready to start
the BGP daemon.

To observe the filter behavior, we recommend that you use [ExaBGP](https://github.com/Exa-Networks/exabgp "ExaBGP GitHub") to generate custom attributes.

Finally, we provide a [Dockerfile](https://github.com/pluginized-protocols/libxbgp/blob/master/misc/Dockerfile_xbgp "xBGP Dockerfile") that includes our modified version of both BIRD and FRRouting, as well as a compiler (clang) to generate eBPF
bytecodes. Currently, you need to manually to start the BGP from inside the docker. We will continue to maintain the docker to
improve its configuration.

