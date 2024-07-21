lets say you use nix-user-chroot and alike things on non nixos hosts or whatever host you lack privilege to use nix like expected
use sth like this

```sh
nix build .# --trusted-substituters  "ssh://username@some.place?remote-program=/home/username/nix-store-custom
```
assuming you use nix-user-chroot ~/nix-store-custom would look sth like:

```sh
LANG="C.UTF-8"
./nix-user-chroot/nix-user-chroot /home/username/nix  /home/username/nix/store/rg0rql48f2h86r819gmyax87knqggx1c-user-environment/bin/nix-store $@
```
__plase note the `$@`!!!!!__ at the end this matters a lot as when nix-store-custom is being called by nix build via ssh it will want to provide the parameters
to the invocation of nix-store thus $@ ensures the parameters get carried over to the actual nix-store invocation running in the user-chroot

if you run a custom patched nix installation you ofc can skip this part and call into the right store dir directly but you likely
know this if you managed to figure out that much already. I use the user-chroot since uhm less compiling since it works for now
