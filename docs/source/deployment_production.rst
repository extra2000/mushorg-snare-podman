Production Deployment
=====================

Example how to deploy Snare for single-instance production environment.

Clone repositories
------------------

.. code-block:: bash

    mkdir ~/extra2000
    cd ~/extra2000
    git clone https://github.com/extra2000/mushorg-snare-podman.git
    git clone https://github.com/extra2000/mushorg-snare.git mushorg-snare-podman/src/snare

Then, ``cd`` into project root directory:

.. code-block:: bash

    cd mushorg-snare-podman

Build snare Image
------------------

From the project root directory, ``cd`` into ``src/snare/`` and then build:

.. code-block:: bash

    cd src/snare/
    podman build -t extra2000/mushorg/snare .

Deploy snare
-------------

From the project root directory, ``cd`` into ``deployment/production/snare``:

.. code-block:: bash

    cd deployment/production/snare

Create config files and fix permissions:

.. code-block:: bash

    cp -v configmaps/snare.yaml{.example,}

Create pod file:

.. code-block:: bash

    cp -v snare-pod.yaml{.example,}

Create SELinux security policy:

.. code-block:: bash

    cp -v selinux/snare_podman.cil{.example,}

Load SELinux security policy:

.. code-block:: bash

    sudo semodule -i selinux/snare_podman.cil /usr/share/udica/templates/base_container.cil

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "snare_podman"

Deploy snare:

.. code-block:: bash

    podman play kube --configmap configmaps/snare.yaml --seccomp-profile-root ./seccomp snare-pod.yaml

Create systemd files to run at startup:

.. code-block:: bash

    mkdir -pv ~/.config/systemd/user
    cd ~/.config/systemd/user
    podman generate systemd --files --name snare-pod-srv01
    systemctl --user enable container-snare-pod-srv01.service
