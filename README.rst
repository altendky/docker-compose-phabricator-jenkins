Overview
========

This Docker composition is an effort to provide an integrated Phabricator_
and Jenkins_ setup for reference.  It presently has umpteen security issues
etc but may still be a useful reference for some of the steps required to get
these two systems working together.

Phabricator_ provides many software development features including VCS
serving and browsing, ticketing, and some tools for build systems.  Jenkins_
is a dedicated build server.  The integration is aided by the
`Uber phabricator-jenkins-plugin`_.

.. _Phabricator: https://www.phacility.com/phabricator/
.. _Jenkins: https://jenkins.io/
.. _Uber phabricator-jenkins-plugin: https://github.com/uber/phabricator-jenkins-plugin

The existing `Phabricator image`_ and `Jenkins image`_ from Bitnami are used
as the base images for this composition.

.. _Phabricator image: https://hub.docker.com/r/bitnami/phabricator/
.. _Jenkins image: https://hub.docker.com/r/bitnami/jenkins/

You may wish to add these lines to your ``/etc/hosts`` file to aid in
accessing the created containers.  Phabricator requires a ``.`` in its name
so it can't be just ``phabricator``.  Jenkins is running on port 8080 so
access it via ``http://jenkins:8080/``

.. code::

  127.0.0.1       jenkins
  127.0.0.1       phabricator.local

This was tested with a pipeline project.  The project itself is private but
the ``Jenkinsfile`` is included here as ``example_Jenkinsfile``.

Setup Steps
===========

These instructions are abbreviated at this point and assume some ability
to navigate Phabricator and Jenkins.


Compose The Images
------------------

To start (depending on your Docker setup, `sudo` may be required):

.. code:: bash

  docker-compose up -d

You can watch the Phabricator container log and wait a couple minutes until the
overseer deamon starts.

The ``login:password`` for Phabricator is ``user:bitnami1`` and for Jenkins
``user:bitnami``.  Please change them or entirely remove those users.

For those new to docker-compose, note that the image names are prefixed by
the project name.  This is derived from the directory containing the
``docker-compose.yml`` file unless overridden on the cli.


Phabricator
-----------

- Create a bot user named ``jenkins``

- Create repository

  - Note: In this case I will use an external repository, often you would
    just have Phabricator host the official repository itself.
  - Select the VCS type
  - Specify a name
  - Optionally specify a callsign
  - If you want to use a 'normal' ``.gitmodules`` file for Git submodules you
    will likely want to specify a short name.  This will provide
    ``/sources/shortname`` URL for cloning which is more compatible with
    other hosting URL layouts.
  - Set built-in URI's to I/O type read-only
  - Create a new URI to the external repository such as
    ``ssh://git@github.com:altendky/st`` with I/O type observe
  - Add credentials if it is a private external repository
  - Activate the repository

- Create a build plan

  - Select commit
  - Add a build step of type ``Make HTTP Request``
  - Set the URI to:
    ``http://jenkins:8080/jenkins/job/${repository.callsign}/buildWithParameters?DIFF_ID=${buildable.diff}&PHID=${target.phid}&COMMIT=${buildable.commit}``
  - Set HTTP Method to POST
  - Add Jenkins credential (to be created in Jenkins later)

    - Name ``phabricator@jenkins``
    - Username: ``phabricator``
    - Password: pick something

  - Set ``When complete`` to wait for message

- Create a herald rule
  - Commits
  - Global
  - Rule name: ``Build Commit``
  - Add the relevant repositories to ``when`` > ``repository`` > ``is any of``
  - Set the Action to ``run build plans`` and select ``Build Commit``


Jenkins
-------

- Install the `Uber phabricator-jenkins-plugin`_

  - Clone ``https://github.com/uber/phabricator-jenkins-plugin``

    - This was tested with ``0ba61e0cc30bc8b0fe5bd5110a60f37df32fcd55``

  - From the clone ``./gradlew assemble``
  - In Jenkins manage plugins and install
    ``./build/libs/phabricator-plugin.hpi`` via the advanced tab and
    ``Upload Plugin``.

- Install then configure "Docker" plugin

  - Manage > Configure > Add new cloud > Docker

    - Name: Host Docker
    - Host URI: ``unix://var/run/docker.sock``
    - Check enabled
    - Test the connection for no errors

- Create a new item

  - Have the name match the Phabricator repositories callsign
  - Select pipeline
  - Select Ok
  - Specify pipeline script from SCM
  - Confugre the appropriate SCM
  - Set the URL to ``ssh://git@phabricator.local/diffusion/THECALLSIGN/the-name.git``
  - In additional behaviours > Add

    - wipeout repository and force clone
    - advanced submodule

      - check recursively update
      - check use credentials from parent

  - Add SSH user name with private key credentials

    - Generate a keypair in Phabricator for the 'jenkins' bot user
    - Username ``git``
    - Select ``Enter directly`` and paste the private key

  - Branch specifier ``${COMMIT}``
  - Uncheck lightweight checkout
  - Advanced

    - Set the name to match the Phabricator repository name

  - Check this project is parametrized
  - Add three string parameters named ``DIFF_ID``, ``PHID``, and ``COMMIT``

  - Save

- Add a new user matching the credentials entered into Phabricator above

- Jenkins > Manage > Configure

  - Disable CSRF protection

    - Yeah...  This is bad but I haven't figured it out yet.

  - Default Phabricator credentials

    - Add
    - Phabricator conduit key
    - URL ``http://phabricator.local``

      - This should be HTTPS, but no SSL certificates setup here yet

    - Description ``jenkins@phabricator.local``
    - Conduit token

      - Generate one for the ``jenkins@phabricator.local`` user over in
        Phabricator

  - Phabricator notifications

    - Select the ``http://phabricator.local`` credentials


Quick Test
==========

Find a commit and run the build plan manually.  Check the build page for a
completed status and in the ``Make HTTP Request`` ``Artifacts`` section
for the Jenkins build URI.
