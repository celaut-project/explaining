## CELAUT Node Service Tutorial

This tutorial will guide you through the necessary steps to understand and execute the demo-service in the CELAUT architecture, exploring its main functionalities of communicating with a node, managing resources, and utilizing child services.


## Step 1: Architecture of demo-service repository

demo-service is a service that interacts with a CELAUT node to:

- Manage resources, adjusting memory usage.
- Use child services that the node can generate and execute in the background.
- Visualize node and service information, such as memory limit and gas consumption.

The structure of demo-service includes configuration files in the `.service` directory, a Dockerfile, and the main code in `start.py`.

- The demo-service code can be found on [Demo Service](https://github.com/celaut-project/demo-service).
- The dependency can be found on [Tiny Service](https://github.com/celaut-project/libraries).

#### Files and Configurations

- `.service/Dockerfile`
  Defines the Docker environment where the service will run. It uses Python 3.11 and installs the necessary dependencies to run a Flask application and the CELAUT node driver.

- `.service/pre-compile.json`
  Specifies the required files, service dependencies, and directory structure that the node will use when compiling the service to a service specification.

- `.service/service.json`
  Defines the technical details of the service, such as the access port (5000), hardware architecture (e.g., aarch64), entry point (`start.py`), and environment variables the service can use.

- `start.py`
  The main code of the service, where the behavior of the service is defined through a Flask web application. This file:
  - Provides an HTML interface where the user can view and adjust node resources.
  - Interacts with child services generated and managed by the node.
  - Exposes various HTTP routes that allow clients to communicate with the service.


## Step 2: Generating the service specification. Compiling the Service

To compile the service on a node, follow these steps:

1. **Compile the Service on the Node**
   Use the command:

   ```
   nodo compile <demo-service-path>/
   ```

   This converts the project structure and files into a binary specification that the node can share and execute on other nodes in the CELAUT network.

2. **Distribution**
   Once compiled, the binary specification of the service can be sent and executed on any node in the network, without the need for additional installations.


## Step 3: Execute the instance of a service

A instance of a service, that is, a container based on a service specification, can be executed following this:

   - If your node has a connection to another node that already has the compiled service specification, you only need to run the following command:
     ```
     nodo execute <id>
     ```
     Replace `<id>` with the ID of the compiled specification that is available on the other node.

   - If your node does not have access to the compiled specification on another node, then you will need to clone the repository and compile the specification on your own node before you can run it. To do this, follow these steps:
     1. Clone the repository containing the `demo-service`.
     2. Compile the service on your node using the command:
        ```
        nodo compile <demo-service-path>/
        ```
     3. Now you can run the service on your node using the command:
        ```
        nodo execute <id>
        ```
        Replace `<id>` with the ID of the specification you have compiled.


## Step 4: Exploring the Functionalities of demo-service

#### User Interface

The demo-service includes a web interface accessible at `http://<node-ip>:5000`, where you can:

- **Manage Memory**
  Adjust the memory limit using the "Increase" and "Decrease" buttons. Send the adjustment to the node using the "Send" button.

- **Generate and Use Child Services**
  Create child services that the node will execute independently. By pressing "Generate New Service", the node creates an instance of the child service specified in `TINY_SERVICE`, which can be executed and monitored.

- **Visualize Resource Consumption**
  The interface shows the real-time memory consumption and the amount of gas available for the service. This information is useful for monitoring the stability of the service and adjusting resources as needed.

  Note: `http://<node-ip>:5000` is not the actual port exposed by the node, but the one exposed by the service instance. The node will create a bridge and assign a free port for external access.


The demo-service uses the [node_controller library](https://github.com/celaut-project/libraries) to interact with the node. Here's how the container (the demo-service instance) performs these interactions:

#### 1. Registering the Child Service in the Library (`tiny_service`)

In the line:

```python
tiny_service = controller.add_service(service_hash=TINY_SERVICE)
```

The child service `TINY_SERVICE` is registered in the node controller library (`node_controller`). Here, the child service is identified by its `service_hash`, which corresponds to a specific hash identifier of the service we want to use. **This action does not interact directly with the node**, but rather prepares the service in the library for later use.

#### 2. Requesting a New Service Instance (`service_uri`)

To generate a new instance of the child service, the `get_instance` method of `tiny_service` is called:

```python
service_uri = tiny_service.get_instance().uri
```

This method requests the node to create an instance of the registered service. The instance generated by the node is exposed through a URI (`service_uri`), which indicates the IP address and port through which this new instance can be accessed. The address of each new instance is added to the `services` list in `demo-service`, and this is displayed in the HTML interface.

#### 3. Modifying Memory Resources

To adjust the amount of memory available for the service, **demo-service** uses the `modify_resources` method of `controller`. This method allows requesting the node to increase or decrease the memory limit for the service based on the desired adjustment:

```python
controller.modify_resources(
    resources={'max': max_mem_limit, 'min': 0}
)
```

In this code:

- `resources` is a dictionary where `max` defines the new memory limit to be set on the node (converted to bytes).
- `min` can be used to indicate a minimum value, but in this case, it is set to `0`.

The node responds with the new `mem_limit` and `gas_amount` values, which are updated in the `resources` and `gas_amount` variables of `demo-service`. These values reflect the updated state of the resources and are displayed in real-time in the HTML interface.

## Video
[![Video tutorial](https://img.youtube.com/vi/kKyeUSQY32E/0.jpg)](https://youtu.be/kKyeUSQY32E)
