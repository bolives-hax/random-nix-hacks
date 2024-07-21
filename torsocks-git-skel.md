
### read the comments!

this is more or less meant for someone that asked me on how to ensure that their
traffic when using git went trough tor ... if you don't know the context and
can't make sense of this this likely isn't meant for you and i woulnd't use it
since if thats the case you don't know what it does. 

This is __NOT__ meant for the public if there is serious interest I may turn this
into a nixos module, but to get this to be useful for more than just git one/me
would have to instead work with a tuntap interface and network namespaces as
seen in iproute2's `ip netns` command set. If you plan to turn this into what
I just described I suggest making use of nixos-router as that in general is a nice fit
for policy based routing. My approach would be to make a network namespace or
populate a secondary routing table. mark the packets in nftables and use ip rule
to send them where they belong. Doable but since there is no demand for this
I won't bother plus im busy with nixos on IBM Z :| ...


```nix
{config,pkgs,...}: let 
    username = "flandre";
in {
  networking.nftables = {
    enable = true;
    tables = {
      fw-outbound-torsocks = /*let
         V use this instead if you manually specified
         the uid via user.users.id or know it for example
         uid 0 is always root due to assumptions the kernel makes
        uid = toString config.users.users.${username}.uid;
      in */ {
        enable = false; #true;
        family = "inet";
        /* type filter hook output priority 0; policy drop; 
        can be used instead if you want to whitelist users rather
        than blacklisting a single one. Note though that you then
        have to make sure that the tor daemon user is excluded from
        the rules. If you have a single user that you want to confine
        to talking trough torsocks its fine*/


        # you can replace skuid with skgid , just note that then nobody -> nogroup
        content = ''
          chain output-torsocks-user {
            type filter hook output priority 0;

            meta skuid ${username} ip daddr != 127.0.0.1 tcp dport != 9050 log prefix \
            "USER ${username} TRIED TO bypass! tor tor, dropping packet" drop
          }
        '';
      };
    };
    # user supplied groups/users are NOT present while nix checks the
    # nftables rules for correctness. Thus we need to replace the usernames
    # with nobody and the groups (if you use groups instead) with nogroup)
    #
    # NOTE!!!: this DOES NOT mean that nobody/nogroup will end up in the actual
    # ruleset they will only end up in the ruleset that is being examined by the
    # nixos nft ruleset checked. To confirm this you can run sth like:
    #
    # sudo nft list table inet  fw-outbound-torsocks
    preCheckRuleset = "sed 's/skuid ${username}/skuid nobody/g' -i ruleset.conf";
  };

  users.users.${username}.packages = with pkgs; [
    (writeScriptBin "git" ''
      ${torsocks}/bin/torsocks ${git}/bin/git "$@"
    '')
  ];
}
```
