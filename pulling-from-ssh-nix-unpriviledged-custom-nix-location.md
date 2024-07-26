lets say you use nix-user-chroot and alike things on non nixos hosts or whatever host you lack privilege to use nix like expected
use sth like this

```sh
# V doesn't seem to work
# nix build .# --trusted-substituters  "ssh://username@some.place?remote-program=/home/username/nix-store-custom
# use the following instead
nix build .#    --substituters "ssh://username@some.place?remote-program=/home/username/nix-store-custom" --no-require-sigs
# ^ depending if ur user is trusted or not you may want to either add ur user to trusted or run it as root or a trusted user
# as this won't run the actual build as root or whatever since it goes trough the nix daemon either way, you can change it
# as soon as you fetched your deps assuming this is just to bypass lengthy rebuilds like I deal with atm porting nixos to s390x
# / ibm system Z mainframes
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
