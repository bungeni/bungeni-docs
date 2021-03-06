Fabric
------

Fabric is a library for simplifying system administration tasks. Fabric is more focused on application level tasks such as deployment.

Install Fabric:

.. code-block:: bash

    $ pip install fabric

The following code will create two tasks that we can use: ``memory_usage`` and
``deploy``. The former will output the memory usage on each machine. The
latter will ssh into each server, cd to our project directory, activate the
virtual environment, pull the newest codebase, and restart the application
server.

::

    from fabric.api import cd, env, prefix, run, task

    env.hosts = ['my_server1', 'my_server2']

    @task
    def memory_usage():
        run('free -m')

    @task
    def deploy():
        with cd('/var/www/project-env/project'):
            with prefix('. ../bin/activate'):
                run('git pull')
                run('touch app.wsgi')

With the previous code saved in a file named fabfile.py, we can check memory
usage with:

.. code-block:: bash

    $ fab memory_usage
    [my_server1] Executing task 'memory'
    [my_server1] run: free -m
    [my_server1] out:              total     used     free   shared  buffers   cached
    [my_server1] out: Mem:          6964     1897     5067        0      166      222
    [my_server1] out: -/+ buffers/cache:     1509     5455
    [my_server1] out: Swap:            0        0        0

    [my_server2] Executing task 'memory'
    [my_server2] run: free -m
    [my_server2] out:              total     used     free   shared  buffers   cached
    [my_server2] out: Mem:          1666      902      764        0      180      572
    [my_server2] out: -/+ buffers/cache:      148     1517
    [my_server2] out: Swap:          895        1      894

and we can deploy with:

.. code-block:: bash

    $ fab deploy

Additional features include parallel execution, interaction with remote
programs, and host grouping.


Bungeni Fabric files
====================

Bungeni fabric deployment files can be found in the following repositories:

    * `https://github.com/bungeni/bungeni-fab <https://github.com/bungeni/bungeni-fab>`_
    *  `http://bungeni-portal.googlecode.com/svn/fabric/branches/ <http://bungeni-portal.googlecode.com/svn/fabric/branches/>`_

