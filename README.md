# Adaptive Video Streaming via CDN and Load Balancing using Mininet

## Team Members
- Arpan Sahu (220020008/EE22BT008)
- Bhavya Bansal (220020017EE22BT017)
- Padagandla Venkata Naga Haripriya (220020039/EE22BT039)
- Vineeta Nihalani (220020057/EE22BT057)
- Ramnaresh Ghusinga (220120017/MC22BT017)

## Motivation

![](https://cdn.mathpix.com/cropped/2025_04_19_2c07376bf983096307cag-01.jpg?height=695&width=945&top_left_y=1176&top_left_x=300)

Video streaming constitutes a significant portion of daily internet usage, encompassing activities like watching movies, participating in online classes, and viewing live events. To ensure a smooth playback experience without excessive loading times or buffering, Content Delivery Networks (CDNs) are employed. CDNs distribute video content from servers geographically closer to users and manage load effectively.

A real-world CDN integrates multiple components, such as DNS servers for directing users to optimal video servers and adaptive video players that adjust quality based on the user's internet connection speed. These features enhance the viewing experience for numerous users.

This project involves building a simplified CDN simulation to understand its core functionalities. The system comprises video servers hosting content at various quality levels, a proxy server that selects the appropriate video quality based on network conditions, and a load balancer to direct users to the most suitable server. Mininet is used to simulate the network environment and evaluate the system's performance under diverse conditions.

This project facilitates learning about real-world streaming services and provides practical experience with networking concepts like adaptive streaming, load balancing, and client-server interaction. The primary goal is to create a functional simulation of a CDN supporting adaptive video streaming and intelligent server selection via load balancing.

## Goals

*   **Implement Adaptive Video Streaming via a CDN Architecture:** Design a system where video content is available from multiple servers, each offering the same video at different bitrates (e.g., low, medium, high quality), simulating adaptive content delivery in CDNs[1].
*   **Develop Video Servers for Bitrate-Based Streaming:** Set up simple HTTP video servers capable of serving pre-encoded video chunks at multiple bitrates, responding to standard HTTP GET requests, and simulating multiple server locations using different ports or IPs[1].
*   **Enable Adaptive Bitrate Selection Using an HTTP Proxy:** Implement an HTTP proxy that intercepts client requests, estimates network throughput, and dynamically selects and forwards requests for video segments with the most appropriate bitrate, ensuring smooth playback under varying bandwidths[1].
*   **Support Multiple Concurrent Clients:** Ensure the proxy can handle multiple simultaneous client connections, adapting the video bitrate individually for each client based on their network conditions[1].
*   **Simulate Load Balancing:** Implement a standalone load balancer simulating basic DNS behavior, assigning clients to video servers using round-robin or geographic logic based on a static configuration, without direct communication with the video servers[1].
*   **Utilize Web Browsers as Clients with Network Throttling:** Use standard web browsers as clients requesting video through the proxy. Employ browser-based network throttling tools to simulate different bandwidth scenarios and observe system adaptation[1].
*   **Deploy and Visualize the System in a Virtual Network Using Mininet:** Use Mininet to create a virtual network encompassing clients, proxy, load balancer, and video servers. This allows for controlled testing, deployment, and visualization of component interactions under configurable network conditions[1].

## Tech Stack

**Video Server**
*   **Backend:** Python (`videoserver.py`, `launch_videoservers.py`)[1].
*   **Frontend:** HTML (`index.html`, `video_player.html`), CSS (`styles.css`), JavaScript[1].
*   **Streaming Library:** `dash.all.min.js` for MPEG-DASH playback[1].
*   **Dependency/Project Management:** `uv` (`uv.lock`), `pyproject.toml`[1].

**Proxy (miProxy) and Load Balancer**
*   **Programming Language:** \(C++\) (`miproxy.cpp`, `loadBalancer.cpp`, `network_utils.cpp`) for performance[1].
*   **Build System:** CMake (`CMakeLists.txt`)[1].

**Mininet Topology Setup**
*   **Virtualization:** Ubuntu Virtual Machine (if host OS is Windows)[1].
*   **Network Simulation:** Mininet for creating virtual network topology[1].
*   **Scripting:** Python 3 to define and customize the Mininet topology[1].
*   **API:** Mininet Python API for programmatic network definition (hosts, switches, links)[1].

## Features Implemented in the Submitted Version

### VIDEOSERVERS

This Python Sanic-based video server streams specific video files[1].

**Key Features:**
*   **Video Serving:** Serves predefined videos ("michigan-hype-video", "tears-of-steel") from `static/videos/<video_name>/<video_file>`[1].
*   **Streaming Support:** Configured for MPEG-DASH using `dash.all.min.js`, enabling adaptive bitrate streaming[1].
*   **Web Interface:** Provides `index.html` listing videos, dynamically generates `video_player.html`, and serves static assets (CSS, JS, favicon)[1].
*   **Multi-Instance Management:** `launch_videoservers.py` script starts multiple server instances on consecutive ports (e.g., 8000, 8001, 8002) and manages graceful shutdown[1].
*   **Technology Stack:** Uses Sanic web server, Python 3.10+, Jinja2 for templating, and Loguru for logging[1].
*   **Logging:** Server and launcher log events (requests, responses, startup/shutdown) to the console[1].
*   **HTTP Handling:** Prevents client-side caching using `Cache-Control: no-store`, `Pragma: no-cache`, `Expires: 0` headers. Includes a POST endpoint `/on-fragment-received` returning HTTP 418 "I'm a teapot", likely for client-side fragment download logic[1].
*   **Error Handling:** Returns 404 Not Found if requested video name is invalid or files don't exist[1].

### miProxy: An Adaptive Bitrate HTTP Proxy

miProxy acts as an intermediary between web clients and video servers, implementing server-side adaptive bitrate (ABR) logic by intercepting and modifying requests based on estimated network throughput. It can connect to a single fixed server or use a load balancer to select servers dynamically[1].

**1. Core HTTP Proxy Functionality**
*   **Server Role:** Listens for incoming client TCP connections on a specified port (`-l` or `--listen-port`)[1].
*   **Client Connection Handling:** Accepts new client connections (`accept()`)[1].
*   **Backend Server Connection:** Establishes a TCP connection to a video server for each client. The server is either static (`-h`/`-p` args, no `-b` flag) or obtained via a load balancer (`-b` flag enabled)[1].
*   **Concurrency:** Uses `select()` to handle I/O from multiple client and server sockets concurrently[1].
*   **Request/Response Forwarding:** Reads client HTTP requests, parses headers (case-insensitive), determines body length (`Content-Length`), and forwards most requests to the corresponding video server socket. Receives responses from the server and forwards them unmodified to the client[1].
*   **Connection Management:** Maintains client-server socket mappings (`g_connections`, `g_client_to_server_map`, etc.). Cleans up connections upon disconnection or error (`cleanup_connection`). Aims for one proxy-to-server connection per client-to-proxy connection[1].

**2. Adaptive Bitrate (ABR) Streaming Logic**
*   **Manifest File Interception (.mpd files):**
    *   Identifies `GET` requests for `.mpd` files[1].
    *   Fetches the full manifest (e.g., `vid.mpd`) from the video server upon the first request for a video (`fetch_and_parse_manifest`)[1].
    *   Parses the `.mpd` using `pugixml`[1].
    *   Caches extracted bitrates globally (`g_video_info_cache`)[1].
    *   Modifies all client `.mpd` requests to point to a version without the bitrate list (e.g., `vid-no-list.mpd`) before forwarding to the server[1].
*   **Throughput Estimation:**
    *   Tracks clients via a unique `X-489-UUID` HTTP header sent by the client[1].
    *   Intercepts `POST` requests to `/on-fragment-received` (these are not forwarded to the video server)[1].
    *   Extracts segment size (`x-fragment-size`), download start time (`x-timestamp-start`), and end time (`x-timestamp-end`) from headers[1].
    *   Calculates segment throughput: \( \text{Throughput (Kbps)} = \frac{\text{Data Size (bytes)} \times 8}{\text{Elapsed Time (ms)}} \)[1].
    *   Maintains a per-client exponentially weighted moving average (EWMA) of throughput: \( T_{cur} = \alpha \times T_{new} + (1 - \alpha) \times T_{cur} \). Smoothing factor \(\alpha\) is configurable (`-a` argument)[1].
    *   Responds directly to the client with HTTP 200 OK for `/on-fragment-received` requests[1].
*   **Bitrate Selection:**
    *   Identifies `GET` requests for video segments (`.m4s` files, excluding audio/init)[1].
    *   Retrieves the client's current EWMA throughput (\(T_{cur}\))[1].
    *   Selects the highest available bitrate (Kbps) such that \( T_{cur} \geq 1.5 \times \text{Bitrate} \)[1].
    *   Defaults to the lowest bitrate if no bitrate satisfies the condition[1].
*   **Video Segment Request Modification:**
    *   Modifies the URI of video segment requests before forwarding, replacing the bitrate part (e.g., `500` in `vid-500-seg-1.m4s`) with the chosen bitrate (e.g., `800`) using `boost::regex` (`modify_m4s_uri`)[1].

**3. Load Balancing Integration**
*   **Dual Modes:** Operates with or without load balancing based on the `-b` or `--balance` flag[1].
*   **No Load Balancing (Default):** Uses the fixed video server specified by `-h`/`-p` arguments[1].
*   **Load Balancing Enabled (`-b`):** Uses `-h`/`-p` to specify the load balancer service address[1].
*   **Load Balancer Query:** For each new client connection, miProxy sends a request to the load balancer (`get_server_from_lb`) using a specific binary protocol (`common/LoadBalancerProtocol.h`). The request includes the client's IP and a unique request ID[1].
*   **Server Assignment:** Receives a response with the assigned video server's IP and port, handling byte order conversion. This server is used for the lifetime of that client connection[1].

**4. Configuration and Logging**
*   **Command-Line Arguments:** Parsed using `cxxopts`:
    *   `-l` / `--listen-port`: Proxy listening port[1].
    *   `-h` / `--hostname`: Video server or Load Balancer IP/hostname[1].
    *   `-p` / `--port`: Video server or Load Balancer port[1].
    *   `-a` / `--alpha`: EWMA smoothing factor (0.0 to 1.0)[1].
    *   `-b` / `--balance`: Enable load balancing mode[1].
*   **Argument Validation:** Checks for required arguments and valid ranges[1].
*   **Logging:** Uses `spdlog` for detailed logging (info, debug, trace levels) matching specified formats for events like startup, connections, disconnections, request forwarding, and throughput calculation[1].

**5. Dependencies**
*   `cxxopts`: Command-line argument parsing[1].
*   `spdlog`: Logging[1].
*   `pugixml`: XML (MPD manifest) parsing[1].
*   `Boost::regex`: URI modification[1].
*   A common library (`LoadBalancerProtocol.h`)[1].
*   Standard C/C++ libraries for networking, strings, containers[1].

### Load Balancer

This load balancer acts as a simplified DNS stand-in, directing `miProxy` instances to backend video servers based on configuration read at startup. It does not communicate directly with the video servers[1].

**1. Communication Protocol**
*   **Interaction:** Listens for requests from `miProxy` using the protocol defined in `cpp/src/common/loadBalancerProtocol.h`[1].
*   **Request (`LoadBalancerRequest`):** Contains client IP address and a unique request ID from `miProxy`[1].
*   **Response (`LoadBalancerResponse`):** Contains assigned video server IP, port, and the echoed request ID[1].
*   **Byte Order:** Handles network byte order conversion for integer fields (port, request ID) using `htonl`, `ntohl`, etc[1].

**2. Operational Modes** (Mutually exclusive, chosen via flags)
*   **Round-Robin Mode (`--rr` or `-r` flag):**
    *   **Strategy:** Distributes requests sequentially across servers listed in a config file (`-s` argument)[1].
    *   **Input File:** Specifies number of servers, then IP and port per line (e.g., `sample_round_robin.txt`)[1].
    *   **Behavior:** Returns the next server in the list for each request, cycling back to the start[1].
*   **Geographic Distance Mode (`--geo` or `-g` flag):**
    *   **Strategy:** Assigns the "closest" server based on lowest path cost in a network graph defined in a config file (`-s` argument)[1].
    *   **Input File:** Describes graph nodes (clients, switches, servers with IPs or `NO_IP`) and weighted links (e.g., `sample_geography.txt`)[1].
    *   **Behavior:** Calculates shortest path cost from the requesting client IP node to all server nodes; returns the IP of the server with the lowest cost path[1].
    *   **Fixed Port:** Always returns port 8000 for the selected server in this mode[1].
    *   **Edge Cases:**
        *   If multiple servers have the same lowest cost, returns the one with the lower node ID[1].
        *   If the client IP is not in the graph or no path exists to any server, closes the socket connection without responding[1].

**3. Server Functionality**
*   **Listening Socket:** Binds to and listens on a TCP port specified by the `-p` argument[1].
*   **Request Handling:** Accepts connections, receives requests, determines the server based on the active mode and config, sends the response (or closes connection in geo failure case)[1].

**4. Configuration and Execution**
*   **Command-Line Arguments:** Parsed using `cxxopts`:
    *   `-p` / `--port`: Listening port (required)[1].
    *   `-g` / `--geo`: Enable Geographic mode[1].
    *   `-r` / `--rr`: Enable Round-Robin mode[1].
    *   `-s` / `--servers`: Path to configuration file (required)[1].
*   **Mode Exclusivity:** Requires exactly one of `-g` or `-r`[1].
*   **Error Checking:** Validates arguments, port range [1024-65535], and mode selection[1].

**5. Logging**
*   **Structured Logging:** Uses `spdlog` for specific informational messages:
    *   Startup: `Load balancer started on port {}`[1].
    *   Request Received: `Received request for client {} with request ID {}` (ID in host byte order)[1].
    *   Response Sent: `Responded to request ID {} with server {}:{}` (ID, port in host byte order)[1].
    *   Request Failure (Geo only): `Failed to fulfill request ID {}` (ID in host byte order)[1].
*   **Log Level Discipline:** Only these messages are printed at `info` level[1].

**6. Dependencies**
*   `spdlog` (logging)[1].
*   `cxxopts` (argument parsing)[1].
*   Shared common library (`loadBalancerProtocol.h`)[1].
*   Does not require `pugixml` or `Boost::regex`[1].

### MININET TOPOLOGY

A custom Mininet topology (`CDNTopo` class) was created for the CDN simulation[1].

**1. Custom and Purpose-Built (`CDNTopo` class)**
*   Inherits from Mininet's `Topo` for full control over network structure via the `build` method[1].

**2. Flexible Scaling (`n`, `num_clients`)**
*   The `build` method is parameterized by `n` (number of video servers) and `num_clients`, allowing easy testing of different scales with defaults (3 servers, 1 client)[1].

**3. Structured Network Segmentation (Switches)**
*   Uses three switches for logical separation:
    *   `s1`: Client aggregation.
    *   `s2`: Central hub connecting proxy and load balancer.
    *   `s3`: Connects backend video servers.

**4. Clear Host Roles and Addressing**
*   Defined hosts with specific roles and IPs in the `10.0.0.0/24` subnet:
    *   Clients (`h_clientX`): Dynamically generated starting at `10.0.0.1`.
    *   Proxy (`h_proxy`): `10.0.0.10` on `s2`.
    *   Load Balancer (`h_lb`): `10.0.0.20` on `s2`.
    *   Video Servers (`h_vsX`): Dynamically created starting at `10.0.0.21` on `s3`.

**5. Realistic Link Characteristics (`TCLink`)**
*   Uses `TCLink` for all connections with specific bandwidth (`bw`), latency (`delay`), and packet loss (`loss`):
    *   Client links: Slower, lossier (`bw=10`, `delay='50ms'`, `loss=1`).
    *   Proxy, LB, Server links: Faster, reliable (`bw=100`, shorter delays).
    *   Switch backbone links: High capacity (`bw=1000`).

**6. Simulated CDN Traffic Path**
*   Connections model the expected request path: Client -> `s1` -> `s2` -> (Proxy/LB) -> `s3` -> Server.

## How to Run

### (1) Testing everything locally:

*   **Requirements:** Ubuntu terminal or Ubuntu via WSL/Virtual Machine on Windows[1].

**Running the Video Server**
1.  Unzip video files if necessary (pulled via Git LFS)[1].
2.  Install the `uv` Python package manager if you haven't already[1].
3.  Navigate to the `videoserver/` directory[1].
4.  Install dependencies: `uv sync`[1].
5.  Launch servers: `uv run launch_videoservers.py`[1].
    *   This script accepts arguments:
        *   `-n` or `--num-servers`: Number of servers to launch (default: 1)[1].
        *   `-p` or `--port`: Starting port (default: 8000). Servers run on sequential ports[1].
    *   Example for 3 servers on ports 8000, 8001, 8002:
        ```
        uv run launch_videoservers.py -n 3 -p 8000
        ```
        ![](https://cdn.mathpix.com/cropped/2025_04_19_2c07376bf983096307cag-17.jpg?height=312&width=1636&top_left_y=695&top_left_x=250)
6.  Access the server index in your browser: `http://127.0.0.1:8000` or `http://localhost:8000`[1].
    ```
    CS 348:COMPUTER NETWORKS PROJECT (SPRING 2025)

    Videos

    Welcome to the CS 348 Project video server! Below you will find links to some example videos:

    > Tears of Steel
    > Michigan Hype Video
    ```
7.  Click video links to play. "Tears of Steel" shows bitrate watermarks[1]. Expected terminal output upon interaction:
    ![](https://cdn.mathpix.com/cropped/2025_04_19_2c07376bf983096307cag-18.jpg?height=812&width=1630&top_left_y=250&top_left_x=253)

**For Launching miProxy and Load Balancer:**

**STEP 1: Running the CMake files**
1.  Install build dependencies:
    *   macOS: `brew install cmake boost`[1].
    *   Ubuntu/WSL: `sudo apt-get update && sudo apt-get install -y cmake libboost-all-dev`[1].
2.  Navigate to the project's main directory[1].
3.  Build the C++ components:
    ```
    mkdir build
    cd build
    cmake ../cpp
    make
    ```
    Expected output includes fetching dependencies and building targets `common`, `miProxy`, `loadBalancer`[1].

**STEP 2: Launching loadBalancer**
*   Assuming local testing uses Round-Robin mode with config `sample_round_robin.txt` and port 9000:
    ```
    cd bin  # Navigate to build/bin directory
    ./loadBalancer --rr -p 9000 -s sample_round_robin.txt
    ```
    Expected output:
    ```
    [2025-04-19 00:24:23.547] [info] Starting in Round-Robin mode using config file: sample_round_robin.txt
    [2025-04-19 00:24:23.550] [info] Successfully parsed 3 servers for Round Robin mode.
    [2025-04-19 00:24:23.551] [info] Load balancer started on port 9000
    ```

**STEP 3: Launch the miProxy**
*   Assuming proxy listens on 8080, connects to load balancer at `127.0.0.1:9000`, uses alpha 0.5, and enables load balancing mode (`-b`):
    ```
    ./miProxy -b -l 8080 -h 127.0.0.1 -p 9000 -a 0.5
    ```
    Expected output:
    ```
    [2025-04-19 00:25:19.437] [info] Debug logging enabled.
    [2025-04-19 00:25:19.439] [info] Configuration: ListenPort=8080, TargetHost=127.0.0.1, TargetPort=9000, Alpha=0.5, LoadBalance=true
    ...
    [2025-04-19 00:25:19.439] [info] miProxy started on port 8080
    ...
    ```
4.  Access the video player through the proxy in your browser: `http://127.0.0.1:8080`[1].
    ![](https://cdn.mathpix.com/cropped/2025_04_19_2c07376bf983096307cag-20.jpg?height=828&width=1630&top_left_y=1207&top_left_x=253)

**Modifying Bandwidth**
*   Use browser developer tools (e.g., Chrome/Firefox: Right Click -> Inspect -> Network tab) to throttle network speed[1].
    ![](https://cdn.mathpix.com/cropped/2025_04_19_2c07376bf983096307cag-21.jpg?height=708&width=1644&top_left_y=405&top_left_x=246)
*   Observe changes in bitrate (via video watermark) and proxy logs (EWMA throughput calculation)[1].
    ![](https://cdn.mathpix.com/cropped/2025_04_19_2c07376bf983096307cag-21.jpg?height=841&width=1635&top_left_y=1225&top_left_x=245)
    ![](https://cdn.mathpix.com/cropped/2025_04_19_2c07376bf983096307cag-22.jpg?height=675&width=1636&top_left_y=248&top_left_x=250)
*   Observe load balancer logs showing round-robin server assignment:
    ```
    [2025-04-19 19:00:00.952] [info] Received request for client 127.0.0.1 with request ID 15082
    [2025-04-19 19:00:00.953] [info] Responded to request ID 15082 with server 127.0.0.1:8000
    [2025-04-19 19:03:18.443] [info] Received request for client 127.0.0.1 with request ID 39389
    [2025-04-19 19:03:18.453] [info] Responded to request ID 39389 with server 127.0.0.1:8001
    [2025-04-19 19:03:19.131] [info] Received request for client 127.0.0.1 with request ID 17307
    [2025-04-19 19:03:19.131] [info] Responded to request ID 17307 with server 127.0.0.1:8002
    ...
    ```

**Please run the files in the order we have specified.**[1]

### WITH MININET:

Testing with Mininet provides a more realistic network simulation and is necessary for testing Geographic load balancing mode[1].

**STEP 1: Launching mininet topology**
1.  Install Mininet:
    ```
    # Install system dependencies
    sudo apt update
    sudo apt install -y git curl python3-pip net-tools

    # Clone the Mininet GitHub repo
    git clone https://github.com/mininet/mininet.git
    cd mininet

    # Run the install script
    sudo ./util/install.sh -a
    ```
2.  Launch the custom topology (assuming 3 video servers, topology file at `~/cn_project/cn_topo.py`):
    ```
    sudo mn --custom ~/cn_project/cn_topo.py --topo cdntopo,3 --link tc --nat
    ```
3.  Start components within Mininet hosts. This can be done via `xterm` or directly in the Mininet CLI.

    **Option A: Using xterm (Example for Geo Mode setup)**
    ```
    mininet> xterm h_vs1 h_vs2 h_vs3 h_lb h_proxy
    ```
    Then, in each respective terminal:
    *   `h_vsX`: `cd path/to/videoserver && uv run videoserver.py 8000` (adjust port/path)
    *   `h_lb`: `cd path/to/build/bin && ./loadBalancer --geo -p 9000 -s path/to/sample_geography.txt` (adjust mode/config)
    *   `h_proxy`: `cd path/to/build/bin && ./miProxy -b -l 8080 -h 10.0.0.20 -p 9000 -a 0.5` (use `h_lb` IP: `10.0.0.20`)

    **Option B: Using Mininet CLI (Example for Round Robin Mode setup)**
    ```
    mininet> h_vs1 cd path/to/videoserver && uv run videoserver.py 8000 &
    mininet> h_vs2 cd path/to/videoserver && uv run videoserver.py 8001 & # Adjust ports as needed
    mininet> h_vs3 cd path/to/videoserver && uv run videoserver.py 8002 &
    mininet> h_lb path/to/build/bin/loadBalancer --rr -p 9000 -s path/to/build/bin/sample_round_robin.txt &
    mininet> h_proxy path/to/build/bin/miProxy -b -l 8080 -h 10.0.0.20 -p 9000 -a 0.5 &
    mininet> h_client1 firefox http://10.0.0.10:8080/ & # Access via proxy IP: 10.0.0.10
    ```
    ![](https://cdn.mathpix.com/cropped/2025_04_19_2c07376bf983096307cag-26.jpg?height=1234&width=1636&top_left_y=245&top_left_x=250)

**Round Robin Mode (Mininet):**
*   Access the proxy from a client inside Mininet (e.g., `h_client1 firefox http://10.0.0.10:8080/`).
*   `miProxy` (on `h_proxy` at `10.0.0.10`) contacts `loadBalancer` (on `h_lb` at `10.0.0.20`).
*   Load balancer returns the next server IP/port (e.g., `10.0.0.21:8000`) based on its list.
*   `miProxy` connects to the assigned video server (e.g., `h_vs1` at `10.0.0.21`). Traffic flows Client -> Proxy -> Server -> Proxy -> Client within Mininet[1].

**Geographic Mode (Mininet):**
*   Start `loadBalancer` with `--geo` and the geography config file (`-s sample_geography.txt`)[1].
    ![](https://cdn.mathpix.com/cropped/2025_04_19_2c07376bf983096307cag-27.jpg?height=1237&width=1638&top_left_y=970&top_left_x=249)
*   Start `miProxy` in load balancing mode (`-b`) pointing to the load balancer's IP (`10.0.0.20`)[1].
*   When a client *inside Mininet* (e.g., `h_client1` with IP `10.0.0.1`) connects to the proxy (`10.0.0.10:8080`), `miProxy` sends the client's IP (`10.0.0.1`) to the load balancer[1].
*   Load balancer finds `10.0.0.1` in its graph config, calculates shortest paths to servers (`10.0.0.21`, `10.0.0.22`, etc.), and returns the closest server's IP (e.g., `10.0.0.21`) and port 8000[1].
    ![](https://cdn.mathpix.com/cropped/2025_04_19_2c07376bf983096307cag-28.jpg?height=2241&width=1633&top_left_y=248&top_left_x=249)
*   `miProxy` connects to the assigned server.

*Note on Geo Mode from Host Browser:* Accessing the proxy (`10.0.0.10:8080`) from your *host machine's browser* will likely fail in Geographic mode. `miProxy` will send your host machine's IP to the load balancer. Since this IP is not defined as a CLIENT node in `sample_geography.txt`, the load balancer will fail to find a path and close the connection, causing errors in `miProxy` and the browser[1]. Geo mode testing requires requests originating from client nodes defined within the Mininet topology and the geography config file.

