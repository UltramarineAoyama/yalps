# Container-native packages

Right now, most package formats are distributed using file archives with their own respective format. But what if... we distribute packages as OCI containers?

## Containers-as-packages

- Each package is simply just a blank OCI file without anything else in root, And to install is to simply just copy the contents of the entire container into root.
- A package will consist of OCI annotations to specify its metadata, and a container without those annotations are considered invalid packages.
- Transactions will be recorded with logs of each container/package applied, and the files that are applied into it.
- The package version will simply be an OCI tag, so a user can just do something like `latest` and pull packages, or pin a package to a specific tag
- (possibly) Instead of copying files to root, we could also make the system immutable by just... layering each package into a temporary root! Building images will simply be a matter of layering each container into 1 single container.
  - ..or do some kind of symlink shenanigans like NixOS.

