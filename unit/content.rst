Official images support initial container configuration, implemented with an
:samp:`ENTRYPOINT` `script
<https://docs.docker.com/engine/reference/builder/#entrypoint>`_.

First, the script checks the Unit :ref:`state directory
<source-config-src-state>` (:file:`/var/lib/unit/` in official images) of
the container.  If it's empty, the script processes certain file types in the
container's :file:`/docker-entrypoint.d/` directory:

.. list-table::
   :header-rows: 1

   * - File Type
     - Purpose/Action

   * - :file:`.pem`
     - :ref:`Certificate bundles <configuration-ssl>` are uploaded under
       respective names: :samp:`cert.pem` to :samp:`certificates/cert`.

   * - :file:`.json`
     - :ref:`Configuration snippets <configuration-mgmt>` are uploaded as
       to the :samp:`config` section of Unit's configuration.

   * - :file:`.sh`
     - :nxt_hint:`Shell scripts <Use shebang in your scripts to specify a
       custom shell>` that run in the container after the :file:`.pem` and
       :file:`.json` files are uploaded.

.. note::

   The script issues warnings about any other file types in the
   :file:`/docker-entrypoint.d/` directory.  Also, your shell
   scripts must have execute permissions set.

This mechanism allows you to customize your containers at startup, reuse
configurations, and automate your workflows, reducing manual effort.  To use
the feature, add :samp:`COPY` directives for certificate bundles, configuration
fragments, and shell scripts to your :file:`Dockerfile` derived from an
official image:

.. subs-code-block:: docker

   FROM nginx/unit:|version|-minimal
   COPY ./*.pem  /docker-entrypoint.d/
   COPY ./*.json /docker-entrypoint.d/
   COPY ./*.sh   /docker-entrypoint.d/

.. note::

   Mind that running Unit even once populates its :samp:`state` directory; this
   prevents the script from executing, so this script-based initialization must
   occur before you run Unit within your derived image.

This feature comes in handy if you want to tie Unit to a certain app
configuration for later use.  For ad-hoc initialization, you can mount a
directory with configuration files to a container at startup:

.. subs-code-block:: console

   $ docker run -d --mount \
            type=bind,src=:nxt_ph:`/path/to/config/files/ <Use a real path instead>`,dst=/docker-entrypoint.d/ \
            nginx/unit:|version|-minimal)

