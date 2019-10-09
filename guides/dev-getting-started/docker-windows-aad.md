# Special notes about Appsody and Docker Desktop on Windows 10

Docker Desktop on Windows requires access to the computer's filesystems in order to mount host volumes contained in those filesystems. That access is explained in detail in the "Shared Drives" section of the [Get started with Docker for Windows](https://docs.docker.com/docker-for-windows/) page.

Appsody relies on the mounted hosted volumes to expedite the startup of applications under development, so that it is imperative that Docker Desktop be configured to access the shared drive containing the user's home directory and that the user associated with the shared drive have the Windows "Full Control" permission on mounted directories.

In most cases, it is sufficient to configure Docker with the same user as the user developing applications with Appsody.

However, for users of Windows 10 Enterprise secured with Azure Active Directory (AAD), the AAD user does not reside in the local host and may not be accepted in the "Shared Drives" tab of the Docker Desktop Settings page, specially if the organization has configured AAD to only issue PIN codes instead of user passwords.

## Workaround for Windows 10 Enterprise secured with Azure Active Directory

If Docker Desktop does not accept your AAD user and password in the "Shared Drives" panel, the only known workaround at this time is to use a separate, local, account to handle the drive sharing and file permissions.

Assuming the creation of a new user does not violate your organization policies, the workaround is comprised of two steps:

1. [Create a new local user account](https://support.microsoft.com/en-us/help/4026923/windows-10-create-a-local-user-or-administrator-account
) on Windows and use that account and respective password in the "Shared Drives" tab. For instance, if you created an account named "Developer", you would have the following settings:

    <img src="images/docker-windows-ad-share-drive.png">

2. Grant that new user the "Full Control" permission to the folders to be mounted by Appsody into a container. You could use the "Security" tab for each folder properties in the File Explorer application, but the quickest and simplest mechanism is to open a "Command Prompt" and issue the following commands:

    ```
    mkdir %USERPROFILE%\.m2\repository
    mkdir %USERPROFILE%\.appsody
    mkdir %USERPROFILE%\workspace\kabanero-workshop

    set DOCKER_SHARED_DRIVE_USER=Developer

    icacls "%USERPROFILE%\.m2" /grant %DOCKER_SHARED_DRIVE_USER%:(OI)(CI)F
    icacls "%USERPROFILE%\.appsody" /grant %DOCKER_SHARED_DRIVE_USER%:(OI)(CI)F
    icacls "%USERPROFILE%\workspace" /grant %DOCKER_SHARED_DRIVE_USER%:(OI)(CI)F
    icacls "%USERPROFILE%\workspace\kabanero-workshop" /grant %DOCKER_SHARED_DRIVE_USER%:(OI)(CI)F
    ```

    Note that the user in `DOCKER_SHARED_DRIVE_USER` must match the user specified in the Docker Desktop "Shared Drives" tab in the first step. 

    Repeat the `mkdir` and `icacls` commands for any other directory where you are about to create a new Appsody project. If you plan on creating multiple projects, you can also target a parent directory for those future project directories, so that you do not have to repeat the commands for each new Appsody project directory.

    Since the parameters for `icacls` specify a recursive authorization, granting the permission directly to the `%USERPROFILE%` directory could be faster and simpler, but the instructions in this page prioritized minimal impact for the workaround.

## Removing the workaround

If you want to revert the permissions later, execute a `icacls ... /remove` command on all affected directories. For instance, for the specific instructions in this page, you would execute the following commands:

```
set DOCKER_SHARED_DRIVE_USER=Developer

icacls "%USERPROFILE%\.m2" /remove %DOCKER_SHARED_DRIVE_USER%
icacls "%USERPROFILE%\.appsody" /remove %DOCKER_SHARED_DRIVE_USER%
icacls "%USERPROFILE%\workspace" /remove %DOCKER_SHARED_DRIVE_USER%
```

You may decide whether or not to remove the directories after removing the authorizations.
