# docker-on-windows-gotchas

These are the lessons learned when trying to get docker v1.12.1 and docker-compose v1.8.0 working on Windows 10.

# Line Endings
Git will convert text files to CRLF on Windows which means the Linux container can fail to build or run.

To force LF line endings for docker related files, you can create `.gitattributes` file in top-level of your repository with the following lines (change as desired):
```
# Ensure shell script uses LF.
Dockerfile  eol=lf
*.sh        eol=lf
```
http://stackoverflow.com/a/34810209/4548096

# File Permissions
There are differences on how docker v1.12.1 manages files on windows vs how docker-compose v1.8.0 does.  

Docker makes all files copied into container executable and provides the following warning:
```
SECURITY WARNING: You are building a Docker image from Windows against a non-Windows Docker host. All files and directories added to build context will have '-rwxr-xr-x' permissions. It is recommended to double check and reset permissions for sensitive files and directories.
```

Docker-compose doesn't do this, so executable files that are checked into git will no longer be executable when using ADD or COPY.  Those files will require a `chmod +x` step to be added to Dockerfile.

The flyway user had to be removed from the flyway container so `chmod +x /cmd.sh || true` could be executed in the child container.

# Other
- Hyper-V enabled breaks VirtualBox: https://www.reddit.com/r/docker/comments/4rvlwp/possible_to_run_virtualbox_and_docker_with_hyper/
- Alpine is not a container to use when debugging issues. Switching to debian to troubleshoot got us better feedback as to what the issues were. We then switched back to alpine image once we got the debian version running.
- There were issues linking schemas image to database image in docker. I could see the database running with `docker ps`, but the schemas couldn't link to it.  This didn't seem to be an issue with docker-compose.
- There is tool called EditorConfig (http://editorconfig.org/) that can be plugged in to IDEs to insure .sh and .sql files are managed in an editor with LF line feeds. It might be worth looking at if this becomes an issue.
  
