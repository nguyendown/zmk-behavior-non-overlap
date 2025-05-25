# Non-overlap Behavior

This behavior allows only one key to be pressed at a time.

## Build

See [Building With Modules](https://zmk.dev/docs/features/modules#building-with-modules).

Edit `zmk-config/config/west.yml`.

```diff
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/zmkfirmware
+    - name: nguyendown
+      url-base: https://github.com/nguyendown
  projects:
    - name: zmk
      remote: zmkfirmware
      revision: main
      import: app/west.yml
+    - name: zmk-behavior-non-overlap
+      remote: nguyendown
+      revision: main
  self:
    path: config
```

## Keymap

The default Non-overlap instance `&nkp` uses key press binding `&kp`.
Any keycode from the usage page should work.

Add this to `.keymap` to use `&nkp`.

```diff
#include <behaviors.dtsi>
+#include <behaviors/non_overlap.dtsi>
#include <dt-bindings/zmk/keys.h>
```

The following keymap prevents A and D from overlapping each other.

```dts
/ {
    keymap {
        default_layer {
            bindings = <
                &kp  Q &kp W &kp  E
                &nkp A &kp S &nkp D>;
        };
    };
};
```

For W and S, create a new Non-overlap instance.
See [Multiple instances](#multiple-instances)

## Config

### `resume-capacity`

Non-overlap behavior tries to resume any remaining pressed keys
in the exact order they were pressed.
`resume-capacity` limits the number of keys this behavior can resume.
The default is 9.

In this example, when you press in sequence ASDF and then release them in reverse order FDSA,
only D is resumed.

```dts
&nkp {
    resume-capacity = <1>;
};

/ {
    keymap {
        default_layer {
            bindings = <
                &nkp A &nkp S
                &nkp D &nkp F>;
        };
    };
};
```

### `no-resume`

When `no-resume` is set, the behavior will not resume any remaining pressed keys
and the property `resume-capacity` is ignored.

```dts
&nkp {
    no-resume;
};

/ {
    keymap {
        ...
    };
};
```

You can also set `resume-capacity = <0>` to achieve the same effect.

## Multiple instances

The following example creates a new Non-overlap instance that works separately from the default `&nkp`.

```dts
/ {
    behaviors {
        nkp2: nkp2 {
            compatible = "zmk,behavior-non-overlap";
            #binding-cells = <1>;
            bindings = <&kp>;
        };
    };

    keymap {
        default_layer {
            bindings = <
                &kp  Q &nkp2 W &kp  E
                &nkp A &nkp2 S &nkp D>;
        };
    };
};
```

## Test

```shell
cd /path/to/zmk/app
ZMK_EXTRA_MODULES="/path/to/zmk-behavior-non-overlap" west test /path/to/zmk-behavior-non-overlap/tests
```

For a specific test case.

```shell
cd /path/to/zmk/app
ZMK_EXTRA_MODULES="/path/to/zmk-behavior-non-overlap" west test /path/to/zmk-behavior-non-overlap/tests/basic
```
