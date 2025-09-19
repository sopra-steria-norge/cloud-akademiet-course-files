# Cloud Akademiet - Dev Box

Obs: As a cost-saving measurement a forced hibernation is run on a schedule.

### Create a new Dev Box

1. Sign into the Developer Portal: https://devportal.microsoft.com/
2. Press **+ New** and select **New Dev Box**
3. Give the Dev Box a name, e.g. `devbox-<firstname>`
4. Do **not** select any customizations
5. Select **Create** to begin provisioning your Dev Box (can take 25-65 minutes)

For additional guide, see [the official docs](https://learn.microsoft.com/en-us/azure/dev-box/quickstart-create-dev-box#create-a-dev-box).

### Use your Dev Box

1. Sign into the Developer Portal: https://devportal.microsoft.com/
2. Choose **Connect via Windows App**
  - If you do not have the Windows App, press the link to download and install this app
    - [MacOS Download](https://apps.apple.com/us/app/windows-app/id1295203466?mt=12)
    - [Windows Download](https://apps.microsoft.com/detail/9n1f85v9t8bn?hl=nb-NO&gl=NO)
3. Press **Connect** to connect to your Dev Box
4. Enter your Sopra Steria username and password to login

For additional guide, see [the official docs](https://learn.microsoft.com/en-us/azure/dev-box/quickstart-create-dev-box#connect-to-a-dev-box)

### First time configuration

Upon the initial boot there will be some installations that will automatically start:

1. Accept the prompt for installing SQL Server 2022 Express
2. Wait for the installation of SQL Server 2022 Express to complete

You can disregard any Sopra Steria-branded software pop-ups (not needed for this workshop).

### Rancher Desktop Installation

You will for a later part of the workshop need to install [Rancher Desktop](https://rancherdesktop.io). To do this:

1. Open the Terminal application
2. Run `winget install SUSE.RancherDesktop` and select 'Y' to accept the installation
3. Accept the administrator prompt to continue the install (there might be two prompts here!)
5. Restart the Dev Box and reconnect after a couple of minutes
4. Open 'Rancher Desktop' (you don't need to enable Kubernetes) and ensure it runs
6. Open the Terminal application and run `docker run hello-world`. It should show the following content:

```
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

### Tips and tricks

- **Restart machine**: In case of required restarts after installations or updates you can restart via Windows menu after connecting, wait a few minutes and then connect again. You can also restart the machine via the Developer Portal.
- **Install missing software**: Use `winget` to install software. See list of required packages in [this script](./scripts/winget.ps1) that should already be installed during provisioning. You can search all available software for Winget [here](https://winstall.app/).
- **Update software**: Use `winget upgrade` to upgrade Software. You can also run Windows Update in settings to trigger system updates.
- **Hibernation**: The machine is set to hibernate after 60m of inactivity. This is to ensure cost efficiency and not to pay for the machine while it is not in use.
- **Visual Studio versions**: The machine already have Visual Studio Professional 2022 installed. If you do not have an license and want to use the community version this will show as "Visual Studio 2022 - Community" in the applications overview.

## Learn more

- [Microsoft Dev Box Documentation](https://learn.microsoft.com/en-us/azure/dev-box/)
- [Dev Box Key Concepts](https://learn.microsoft.com/en-us/azure/dev-box/concept-dev-box-concepts)
- [Dev Box Customization](https://learn.microsoft.com/en-us/azure/dev-box/how-to-customize-dev-box-setup-tasks)
