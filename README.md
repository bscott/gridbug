# GridBug
This is a simple tool to monitor network connectivity between nodes and display status graph.

<img width="489" alt="image" src="https://user-images.githubusercontent.com/836718/193515045-d705c6d2-6918-449d-bb98-4e1ee0e98a0d.png">

## How it Works
The `gridbug.py` service pulls in a list of other gridbug nodes. It then proceeds to poll each of these nodes and records the state (link up or down).  

One of the gridbug nodes can be designated as the server and the other grid nodes can send their results to that node. However, every node attempts to converge on the same graph by sending updates and polling each other. Each node builds a directional graph of connectiity between all the gridbug nodes and renders a HTML page using the [cytoscape](https://cytoscape.org/) JavaScript visualization library.

## Configuration

### Environmental Settings

* GRIDBUGCONF = Path to gridbug.conf config file
* GRIDBUGLIST = Path to gridbugs.json node list
* BUGLISTURL = URL to gridbugs.json (overrides above)

## Quick Start

1. Create a `gridbug.conf` file (example below) and update with your specific location details. Make sure you update `ID` to be the unique name of the node.  

2. The `GRIDKEY` token should be a unique key (string of alphanumeric characters). It should be the same for all the GridBug nodes in your network.  GridBug will reject any updates from nodes that do not match this key.

* gridbug.conf - Configuration File
    ```conf
    [GRIDBUG]
    DEBUG = no
    ID = localhost
    NODEURL = example.com:8777
    ROLE = node
    CONSOLE = gridbug.html
    SERVERNODE = localhost:8777
    GRIDKEY = CRuphacroN2hOfachlsWipaxi4ude1rlbIn4v0Vispiho7tuWeSPADrUdR2pE0rl

    [API]
    # Port for API requests
    ENABLE = yes
    PORT = 8777

    [BUGS]
    POLL = 10
    TTL = 30
    TIMEOUT = 10

    [ALERT]
    # Notify connectivity issues - TODO
    ENABLE = yes
    ```                             

3. Create a `gridbugs.json` seed file and add some nodes for your grid. The `host` is the address and should include the port (e.g. `:8777`) where `id` is the unique name of the node (matching gridbug.conf `ID`) for each node.

    * This does not need to be the complete list of GridBugs in your network as GridBug will propagate discovered notes across the grid. If `host` and `id` are missing for the local node, it will automatically add `ID` and `NODEURL` from above config.

    ```json
    {
        "version": 1,
        "gridbugs": [{
            "host": "ptr.example.com:8777",
            "id": "jasonacox.com"
        }, {
            "host": "10.0.1.2:8777",
            "id": "LAN"
        }, {
            "host": "localhost:8777",
            "id": "origin"
        }]
    }
    ```

4. Run the Docker Container to listen on port 8777.

    ```bash
    docker run \
    -d \
    -p 8777:8777 \
    -e GRIDBUGCONF='/var/lib/gridbug/gridbug.conf' \
    -e GRIDBUGLIST='/var/lib/gridbug/gridbugs.json' \
    -v ${PWD}:/var/lib/gridbug \
    --name gridbug \
    --restart unless-stopped \
    jasonacox/gridbug
    ```

5. View the GridBug Console and API Calls

    Console: http://localhost:8777/

    ```bash
    # Get Version and Status
    curl -i http://localhost:8777/stats

    # Get Current Weather Data
    curl -i http://localhost:8777/text   # Text version of console
    curl -i http://localhost:8777/bugs   # List of gridbug nodes
    curl -i http://localhost:8777/graph  # nternal graph of connectivity (JSON)
    ```

### Direct Running

Alternatively, you can just run the python server from a startup script:

```bash
python3 gridbug.py
```

