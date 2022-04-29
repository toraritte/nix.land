`derivation-mysteries/jfgrx01kcv7r95n06hyzn9i7p8qlsmz6-hello-2.10.drv` has been pulled out of my current Nix store (`/nix/store`) and it looks like this:

```
Derive(
  [ ("out","/nix/store/wmsbkyp09jbgp4p9ki62sb44b7ly418v-hello-2.10","","")
  ]
, [ ("/nix/store/75z5ks49cl5apykcnwwhijv3ghndd2sh-hello-2.10.tar.gz.drv",["out"])
  , ("/nix/store/a79xrwasi9fla2pw1hznp76w2h18mvkq-stdenv-linux.drv",["out"])
  , ("/nix/store/hxpwg4g3j7xgk4fxja4ppgy24skqajd5-bash-4.4-p23.drv",["out"])
  ]
, ["/nix/store/9krlzvny65gdc8s7kpb6lkx8cd02c25b-default-builder.sh"]
, "x86_64-linux"
, "/nix/store/vnyfysaya7sblgdyvqjkrjbrb0cy11jf-bash-4.4-p23/bin/bash"
, ["-e","/nix/store/9krlzvny65gdc8s7kpb6lkx8cd02c25b-default-builder.sh"]
, [ ("buildInputs","")
  , ("builder","/nix/store/vnyfysaya7sblgdyvqjkrjbrb0cy11jf-bash-4.4-p23/bin/bash")
  , ("configureFlags","")
  , ("depsBuildBuild","")
  , ("depsBuildBuildPropagated","")
  , ("depsBuildTarget","")
  , ("depsBuildTargetPropagated","")
  , ("depsHostHost","")
  , ("depsHostHostPropagated","")
  , ("depsTargetTarget","")
  , ("depsTargetTargetPropagated","")
  , ("doCheck","1")
  , ("doInstallCheck","")
  , ("name","hello-2.10")
  , ("nativeBuildInputs","")
  , ("out","/nix/store/wmsbkyp09jbgp4p9ki62sb44b7ly418v-hello-2.10")
  , ("outputs","out")
  , ("patches","")
  , ("pname","hello")
  , ("propagatedBuildInputs","")
  , ("propagatedNativeBuildInputs","")
  , ("src","/nix/store/3x7dwzq014bblazs7kq20p9hyzz0qh8g-hello-2.10.tar.gz")
  , ("stdenv","/nix/store/npdq5pilrfsklvhx0ws44lykc1rl1i66-stdenv-linux")
  , ("strictDeps","")
  , ("system","x86_64-linux")
  , ("version","2.10")
  ]
)
```

So nothing like it is described in Dolstra's thesis (i.e., being a valid Nix language expression of an attribute set). [This post](http://blog.tpleyer.de/posts/2020-01-17-nix-show-derivation-is-your-friend.html) introduces `nix show-derivation` (as no official sources really do that), and its output is indeed what I would have expected:

```
$ nix show-derivation /nix/store/jfgrx01kcv7r95n06hyzn9i7p8qlsmz6-hello-2.10.drv

{
  "/nix/store/jfgrx01kcv7r95n06hyzn9i7p8qlsmz6-hello-2.10.drv": {
    "outputs": {
      "out": {
        "path": "/nix/store/wmsbkyp09jbgp4p9ki62sb44b7ly418v-hello-2.10"
      }
    },
    "inputSrcs": [
      "/nix/store/9krlzvny65gdc8s7kpb6lkx8cd02c25b-default-builder.sh"
    ],
    "inputDrvs": {
      "/nix/store/75z5ks49cl5apykcnwwhijv3ghndd2sh-hello-2.10.tar.gz.drv": [
        "out"
      ],
      "/nix/store/a79xrwasi9fla2pw1hznp76w2h18mvkq-stdenv-linux.drv": [
        "out"
      ],
      "/nix/store/hxpwg4g3j7xgk4fxja4ppgy24skqajd5-bash-4.4-p23.drv": [
        "out"
      ]
    },
    "platform": "x86_64-linux",
    "builder": "/nix/store/vnyfysaya7sblgdyvqjkrjbrb0cy11jf-bash-4.4-p23/bin/bash",
    "args": [
      "-e",
      "/nix/store/9krlzvny65gdc8s7kpb6lkx8cd02c25b-default-builder.sh"
    ],
    "env": {
      "buildInputs": "",
      "builder": "/nix/store/vnyfysaya7sblgdyvqjkrjbrb0cy11jf-bash-4.4-p23/bin/bash",
      "configureFlags": "",
      "depsBuildBuild": "",
      "depsBuildBuildPropagated": "",
      "depsBuildTarget": "",
      "depsBuildTargetPropagated": "",
      "depsHostHost": "",
      "depsHostHostPropagated": "",
      "depsTargetTarget": "",
      "depsTargetTargetPropagated": "",
      "doCheck": "1",
      "doInstallCheck": "",
      "name": "hello-2.10",
      "nativeBuildInputs": "",
      "out": "/nix/store/wmsbkyp09jbgp4p9ki62sb44b7ly418v-hello-2.10",
      "outputs": "out",
      "patches": "",
      "pname": "hello",
      "propagatedBuildInputs": "",
      "propagatedNativeBuildInputs": "",
      "src": "/nix/store/3x7dwzq014bblazs7kq20p9hyzz0qh8g-hello-2.10.tar.gz",
      "stdenv": "/nix/store/npdq5pilrfsklvhx0ws44lykc1rl1i66-stdenv-linux",
      "strictDeps": "",
      "system": "x86_64-linux",
      "version": "2.10"
    }
  }
}
```
