# NixOS Docker Compose Module

NixOS module to manage Docker Compose services.

Using this module, installing a service through a Docker Compose file is as easy as pointing to its directory with `dockerComposeServices.composeDirs` from your `configuration.nix`.

Everything alongside the compose file will also be copied and installed, even `build` with a `Dockerfile` works.

## Usage

In your system's `flake.nix`, add this repository as an input:

```nix
# your-system/flake.nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";

    # Add this flake as an input
    docker-compose-services.url = "github:benjamingwynn/docker-compose-services.nix";
  };

  outputs = { self, nixpkgs, docker-compose-services }: {
    nixosConfigurations.your-hostname = nixpkgs.lib.nixosSystem {
      system = "x86_64-linux";
      modules = [
        # Import the module here
        docker-compose-services.nixosModules.default

        # Your main configuration file
        ./configuration.nix
      ];
    };
  };
}
```

Then, in your `configuration.nix`, use the `dockerComposeServices.composeDirs` option:

```nix
# your-system/configuration.nix
{ config, pkgs, ... }:

{
  # ... your other system settings

  # Define the directories containing your docker-compose.yml or compose.yml files.
  # The path should be relative to this file or an absolute path.
  dockerComposeServices.composeDirs = [
    ./my-app-1
    ./my-app-2
    # /path/to/another/project
  ];
}
```

### Config options

By default, the systemd services that manage your Docker Compose projects run as `root`. You can change this by setting the `dockerComposeServices.user` option, make sure the user you select has the `docker` group.

No other configuration options are provided at the moment.

## Comparison with `compose2nix`

While [`compose2nix`](https://github.com/aksiksi/compose2nix) aim to convert your `docker-compose.yml` files into Nix derivations of OCI containers, this instead just allows you to drop your existing `docker-compose.yml` and whatever `Dockerfile` or other config files you have.

The idea is, by not converting your `docker-compose.yml` files to a new format, we avoid "Nix lock-in" and keep our configs portable with other platforms, like Docker Desktop.

This makes it ideal if you prefer to keep your compose files as-is, and simply need a robust NixOS-native way to manage their declaration.
