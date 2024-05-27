######################################################################
# From Makefile to Justfile (or Taskfile): Recipe Runner Replacement #
######################################################################

# Additional Info:
# - just: https://github.com/casey/just
# - Say Goodbye to Makefile - Use Taskfile to Manage Tasks in CI/CD Pipelines and Locally: https://youtu.be/Z7EnwBaJzCk

#########
# Intro #
#########

just cluster-create

just test-watch

just cluster-destroy

#########
# Setup #
#########

git clone https://github.com/vfarcic/crossplane-kubernetes

cd crossplane-kubernetes

git pull

git checkout just

# Make sure that Docker is up-and-running. We'll use it to create a Kind cluster.

# Watch https://youtu.be/WiFLtcBvGMU if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

devbox shell

########################################
# just Define Justfile and just Run It #
########################################

just cluster-create

just test-watch

# Press `ctrl+c` to stop the watcher.

just cluster-destroy

just

just --show cluster-create

cat Justfile

###########
# Destroy #
###########

exit

git checkout main
