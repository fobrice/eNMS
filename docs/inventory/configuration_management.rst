========================
Configuration Management
========================

eNMS can work as a network device configuration backup tool and replace Oxidized/Rancid with the following features:
  - Poll network elements download configurations when they change
  - Easily view current configuration of a device in the inventory
  - Search for any text in any configuration
  - View differences between various revisions of a configuration
  - Download device configuration to a local text file
  - Use the ReST API support to return a specified device's configuration
  - Export all device configurations to a remote Git repository (e.g. Gitlab)

Device configuration
--------------------

All devices are listed in the :guilabel:`inventory/configuration_management` page. By default, configurations are retrieved by a service called ``Configuration Backup Service``, which
  - Uses Netmiko to fetch the configuration
  - Update the device ``configuration`` property (a python dictionnary that contains the most recent configurations)
  - Write the configuration to a local text file (located in eNMS/git/configurations)

.. image:: /_static/inventory/configuration_management/device_configuration.png
   :alt: Configuration Management table.
   :align: center

For some device, the configuration cannot be retrieved only with a netmiko command. You can create your own configuration backup services if need be. Targets are defined at service, like any other services.
The task ``poller_task`` will periodically run all services whose ``configuration_backup_service`` parameter is set to ``True``, like for the default configuration backup service here: https://github.com/afourmy/eNMS/blob/master/eNMS/automation/services/configuration_management/configuration_backup_service.py#L15 

Configure polling
-----------------

The polling process is controlled by the ``Poller`` task. The ``Poller`` task is configured to run the ``Configuration Management Workflow``.

.. image:: /_static/inventory/configuration_management/configuration_management_workflow.png
   :alt: Configuration Management Workflow.
   :align: center

To start the configuration polling mechanism, you need to go to the :guilabel:`scheduling/task_management` page and start (``Resume`` button) the ``Poller`` task.
By default, the ``Poller`` task will run every hour (3600 seconds), but you can change the frequency from the ``Edit`` form.

Search and display the configuration
------------------------------------

You can search for a specific word in the current configuration of all devices with the input field on top of the ``Configuration`` column. eNMS will filter the list of devices based on whether the current configuration of the device contains this word.

By clicking on the ``Configuration`` button, you can display and compare the device configurations.

.. image:: /_static/inventory/configuration_management/display_configuration.png
   :alt: Display Configuration.
   :align: center

All runs are stored in the ``Display`` and ``Compare With`` pull-down lists:
  - Selecting a run from ``Display`` will display the associated configuration.
  - Selecting a run from ``Compare With`` will compare the configuration with the one selected in ``Display``.

Additionally, you can click on ``Raw logs`` to open a pop up that contains nothing but the configuration (useful for copy/pasting), and click on ``Clear`` to remove all previously stored configurations from the database.

Comparing two configurations will display a git-like line-by-line diff similar to the one below:

.. image:: /_static/inventory/configuration_management/compare_configurations.png
   :alt: Compare Configurations.
   :align: center

Advanced
--------

- Number of configurations stored in the database: by default, eNMS stores the 10 most recent configurations in the database. The polling process is controlled by the ``configuration_backup`` service. You can change the number of stored configuration by changing the ``Number of configurations stored`` property.
- Configurations are retrieved with netmiko. By default, eNMS uses the driver defined at device level to run the command. You can use a driver configured at service level instead, by unticking the ``Use driver from device`` check box.
