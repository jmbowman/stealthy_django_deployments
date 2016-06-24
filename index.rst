.. Stealthy Django Deployments presentation master file, created by
   sphinx-quickstart on Sun Jun 19 15:03:03 2016.

===========================
Stealthy Django Deployments
===========================

.. revealjs:: Stealthy Django Deployments
 :subtitle: How to update a Django site without anybody noticing you did it
 :data-transition: slide
 :title-heading: h3
 :subtitle-heading: h4

 | Jeremy Bowman
 | jbowman@edx.org

.. revealjs:: Traditional deployment
 :data-transition: slide
 :title-heading: h3

 .. blockdiag::
  :align: center
  :scale: 150%
 
  diagram {
    orientation = portrait
    "Update site" [color = red];
    "Site is running" -> "Shut site off" -> "Update site" -> "Restart site"
  }

.. revealjs:: Why the need for stealth?
 :data-transition: slide
 :title-heading: h3

 * Minimize inconvenience to users
 * Reduce site errors during deployment
 * Deploy when other people are around to help
 * Enable frequent deployments

.. revealjs:: Main points to consider
 :data-transition: slide
 :title-heading: h3

 * In-progress requests
 * Asynchronous tasks
 * Package installation, upgrades, and removal
 * Database migrations
 * Static assets
 * Configuration changes

.. revealjs:: Add or reload WSGI workers
 :data-transition: slide
 :title-heading: h3

 * Wait until all the new code is in place
 * Use WSGI reload instead of restart
 * Old and new code must be able to run concurrently

.. revealjs:: Restart asynchronous workers
 :data-transition: slide
 :title-heading: h3

 * Stop Celery, etc. before making any real changes
 * Limit the duration of asynchronous tasks
 * Allow some time for tasks to wrap up
 * Terminate workers if shutdown takes too long
 * Restart at end of deployment

.. revealjs:: Install or upgrade dependencies
 :data-transition: slide
 :title-heading: h3

 * Use wheels if at all possible
 * Keep an "uninstall.txt" file
 * Upload package files before installation

.. revealjs:: Database migrations
 :data-transition: slide
 :title-heading: h3

 * Always backwards-compatible
 * Sometimes requires multiple deployments
 * Test on a production-scale database when possible
 * Watch out for deadlock risks

.. revealjs:: What is a backwards-compatible migration?
 :data-transition: slide
 :title-heading: h3

 * Don't remove or dramatically change an in-use field
 * Don't remove an in-use table
 * Don't directly rename an in-use field
 * New fields must initially be nullable

 .. rst-class:: fragment

  Automate these checks whenever possible!

.. revealjs:: Understand your database
 :data-transition: slide
 :title-heading: h3

 * PostgreSQL: Avoid new fields with default values
 * MySQL: Schema changes are slow on any large table
 * MySQL: No transactional schema changes

.. revealjs:: Static assets
 :data-transition: slide
 :title-heading: h3

 * Append hashes to filenames
 * New files must be available before WSGI reloads
 * Don't stop serving the old files

.. revealjs:: In-place stealthy deployment
 :data-transition: slide
 :title-heading: h3

 .. blockdiag::
  :align: center
  :scale: 150%
 
  diagram {
    "Upload new code" -> "Stop async workers" -> "Upgrade dependencies" -> "Run migrations" -> "Update deployment link" -> "Update configuration" -> "Reload WSGI" -> "Start async workers";
    "Upgrade dependencies" -> "Run migrations" [folded];
    "Update configuration" -> "Reload WSGI" [folded];
  }

.. revealjs:: Blue-green deployment
 :data-transition: none
 :title-heading: h3

 .. blockdiag::
  :align: center
  :scale: 150%
 
  diagram {
    "Old cluster" [color = lightblue];
    "Traffic" -> "Old cluster" -> "Database(s)"
  }

.. revealjs:: Blue-green deployment
 :data-transition: none
 :title-heading: h3

 .. blockdiag::
  :align: center
  :scale: 150%
 
  diagram {
    "Old cluster" [color = lightblue];
    "New cluster" [color = green];
    "New cluster" -> "Database(s)";
    "Traffic" -> "Old cluster" -> "Database(s)";
    group {
      color = "none";
      "Old cluster"; "New cluster";
    }
  }

.. revealjs:: Blue-green deployment
 :data-transition: none
 :title-heading: h3

 .. blockdiag::
  :align: center
  :scale: 150%
 
  diagram {
    "Old cluster" [color = lightblue];
    "New cluster" [color = green];
    "Old cluster" -> "Database(s)";
    "Traffic" -> "New cluster" -> "Database(s)";
    group {
      color = "none";
      "Old cluster"; "New cluster";
    }
  }

.. revealjs:: Blue-green deployment
 :data-transition: none
 :title-heading: h3

 .. blockdiag::
  :align: center
  :scale: 150%
 
  diagram {
    "New cluster" [color = green];
    "Traffic" -> "New cluster" -> "Database(s)"
  }

 .. rst-class:: fragment

  * Database migrations **must** be backwards compatible

.. revealjs:: Rolling deployment
 :data-transition: none
 :title-heading: h3

 .. blockdiag::
  :align: center
  :scale: 150%
 
  diagram {
    "Traffic" -> "Node 1" -> "Database(s)";
    "Traffic" -> "Node 2" -> "Database(s)";
    "Traffic" -> "Node 3" -> "Database(s)";
  }

.. revealjs:: Rolling deployment
 :data-transition: none
 :title-heading: h3

 .. blockdiag::
  :align: center
  :scale: 150%
 
  diagram {
    "New node" [color = green];
    "Traffic" -> "Node 1" -> "Database(s)";
    "Traffic" -> "Node 2" -> "Database(s)";
    "Traffic" -> "Node 3" -> "Database(s)";
    "New node" -> "Database(s)";
    group {
      color = "none";
      "Node 1"; "Node 2"; "Node 3"; "New node";
    }
  }

.. revealjs:: Rolling deployment
 :data-transition: none
 :title-heading: h3

 .. blockdiag::
  :align: center
  :scale: 150%
 
  diagram {
    "New node" [color = green];
    "Node 1" -> "Database(s)";
    "Traffic" -> "Node 2" -> "Database(s)";
    "Traffic" -> "Node 3" -> "Database(s)";
    "Traffic" -> "New node" -> "Database(s)";
    group {
      color = "none";
      "Node 1"; "Node 2"; "Node 3"; "New node";
    }
  }

.. revealjs:: Rolling deployment
 :data-transition: none
 :title-heading: h3

 .. blockdiag::
  :align: center
  :scale: 150%
 
  diagram {
    "New node" [color = green];
    "Traffic" -> "Node 2" -> "Database(s)";
    "Traffic" -> "Node 3" -> "Database(s)";
    "Traffic" -> "New node" -> "Database(s)";
  }

.. revealjs:: Other tips
 :data-transition: slide
 :title-heading: h3

 * Automate, automate, automate
 * Test the deployment process in staging
 * Maintain backwards compatibility with other services
 
.. revealjs:: Thank you!
 :data-transition: slide
 :title-heading: h3

 Questions?

 .. rv_small::

  | Jeremy Bowman
  | jbowman@edx.org
