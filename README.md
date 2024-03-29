<div align="center">
  <p>
    <a align="center" href="https://www.hs-kempten.de/ifm" target="_blank">
      <img width="100%" src="docs/FROS_V3.png"></a>
  </p>

<!-- Badges -->
![C++](https://img.shields.io/badge/c++-blue.svg)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://opensource.org/licenses/MIT)
![Contributions welcome](https://img.shields.io/badge/contributions-welcome-orange.svg)
</div>

# [Introduction](#introduction)

**FRos** is a node framework to integrate a [Fieldbus](https://en.wikipedia.org/wiki/Fieldbus) into the [Robot Operating System (ROS)](https://www.ros.org/). 
This project depends on the use of an [Ixxat FRC-EP170](https://www.ixxat.com/products/automotive-solutions/overview/embedded-platform/frc-ep-170?ordercode=1.01.0142.00000) or [Ixxat FRC-EP190](https://www.ixxat.com/products/automotive-solutions/overview/embedded-platform/frc-ep-190?ordercode=1.13.0094.00000) gateway device, which to our knowledge are one of the few gateway devices on the market that can be easily integrated into a ROS environment. ROS is widely used for research and in commercial products in the field of robotics. It allows an easy integration of different sensor types into a common environment and due to its open source nature, many different software packages and drivers are already openly available.

In order to extend the ecosystem beyond the widely used CAN and Ethernet standard to include FlexRay in ROS, we provide this framework for the currently available Ixxat gateways [Ixxat FRC-EP170/190](https://www.ixxat.com/products/automotive-solutions/overview/embedded-platform/frc-ep-190?ordercode=1.13.0094.00000). This node handles all the functionalities for interacting with the device and publishes the received messages as topics in ROS. 
### [Physical requirements](#physical-requirements)
- Gateway device
- Fieldbus definition file (e.g. **.xml* / **.dbc*)
- Ethernet port on your computer

### [Supported Fieldbus systems](#supported-fieldbus-systems)
- [FlexRay](https://www.ixxat.com/technologies/automotive/flexray)
- [CAN](https://www.ixxat.com/technologies/fieldbuses/can)

---

# [Table of contents](#table-of-contents)
- [Introduction](#introduction)
    - [Physical requirements](#physical-requirements)
    - [Supported Fieldbus systems](#supported-fieldbus-systems)
- [Table of contents](#table-of-contents)
- [Process](#process)
  - [Modules](#modules)
    - [ROS-node](#ros-node)
    - [C-coderdbc](#c-coderdbc)
  - [Software requirements](#software-requirements)
  - [Workflow](#workflow)
    - [ACT Setup](#act-setup)
    - [Channel Mapping](#channel-mapping)
  - [Start ROS node](#start-ros-node)
- [Known issues](#known-issues)
- [License](#license)
- [Contact](#contact)
- [Cite](#cite)
---

# [Process](#process)
The following section will give you a fundamentel overview of the project and its single modules. We will also provide you with information to get you own project running. In the area of research, oftentimes the necessary tools need to be developed and be available fast and the important role of tools is overshadowed by potential results. The requirements for a research project may change and the adaption and expansion of the previous research topic needs to happen fast. Thus, the here presented framework is built with auto code generation in mind. A provided **.dbc* network description is used to automatically generate all the relevant code snippets and files.

[[↑ to top ↑](#table-of-contents)]

---

## [Modules](#modules)
The module FRos currently consists of:
### [ROS-node](#ros-node)
The framework itself ([ros-node](./ros-node/)) for ROS that handles the connection to the device and the ROS publishers.

### [C-coderdbc](#c-coderdbc)
 C-coderdbc ([c-coderdbc](./c-coderdbc/)) as a fork of the c-coderdbc by github.com user [astand](https://github.com/astand). This program was created to output C source code from a given **.dbc* file. It has been modified for this use case to output:
  * a more elegant C code based on C structs and bitfields
  * copy-paste ready code snippets to handle the initialization of ROS publishers, to call the appropriate functions based on the message ID and also to generate the necessary **.msg* files for ROS.

The FRos framework is setup as such, that the relevant code snippets can be easily integrated.

<div align="center">
  <img src = "docs/FROS-schema.svg" alt="FRos framework schema" width="75%" />
</div>

[[↑ to top ↑](#table-of-contents)]

---

## [Software requirements](#software-requirements)
To follow the process steps, we recommend the following setup:
- `Linux Ubuntu 20.04.5 LTS`
- `IXXAT ACT 6 (on Windows)`
- `ROS noetic`

## [Workflow](#workflow)
To integrate the Ixxat FRC into ROS, we will demonstrate the workflow in the following by using CAN as fieldbus. Therefore, we use the public available configuration file from [commaai](https://github.com/commaai/opendbc/blob/d585a9bf2908b2c83bf02b567b9e1f5bfc587a01/vw_mqb_2010.dbc) for demonstration purposes. We adapted this software project to work with two specific possible configurations. One configuration maps a selection of whole frames to GenEthernet, the other maps a selection of whole PDUs to GenEthernet. A third possible configuration, the mapping of all frames to GenEthernet, is not included in this release. This would require a different processing of the message ID.

Generally, the customer support and engineers at *HMS* will be able to assist you with your project plans. In the following paragraph, we can provide you with hints how to setup your environment so the outcome fits best for you.

### [ACT Setup](#act-setup)
Open Ixxat ACT and:

1. Select __New project__ and

2. Select __Gateway__ 

![](/docs/ACT_new_project.png) 

3. Select your device 

![](/docs/ACT_device_selection.png)

4. Add GenEthernet to your project

![](/docs/ACT_Add_Gen_Ethernet.png)

5. Keep file description empty as for now and continue with OK

![](/docs/ACT_keep_empty.png)

6. Ensure the following settings: 
- __Remote device__: "FRC Device"
- __Role__: Slave
- __Optimize for__: Neutral 
- __Interface__: LAN 
- __Port__: 19227

![](/docs/ACT_GenEth_settings.png)

7. Add database file ( **.dbc*, **.xml*, ...) to e.g. CAN-1 (since we use CAN (`vw_mqb_2010.dbc` from [commaai](https://github.com/commaai/opendbc/blob/d585a9bf2908b2c83bf02b567b9e1f5bfc587a01/vw_mqb_2010.dbc)) as workflow example) or **.xml* to FR-1 A / B. This depends on your use case and available fieldbus.

![](/docs/ACT_GenEth_final.png)

8. Click "next" to continue

9. Navigate to Tools > Options > Gateway and activate __Adopt Source ID__

![](/docs/ACT_adopt_ID.png)

Now the project is configured and we are able to map the data from Bus to Generic Ethernet (GenEthernet)
### [Channel Mapping](#channel-mapping)
To apply the mapping, follow the following steps: 

10. Select __GenEthernet__ [1]
11. Select __Messages__ [2]
12. On the left column select __Messages__ [3]

![](/docs/ACT_ChannelMapping_1.png)

>Hint: If FlexRay is your fieldbus, you have to generate a **.dbc* file from **.xml* by using `Flx2CanDb`, which will be provided to you by IXXAT on request. 

Apply mapping: 

13. Drag & Drop the messages from left onto GenEthernet on right side (see image above how it could look like)

14. Build configuration ![](/docs/build_config.png)

15. Inside the project folder, the file `project_dir/DeviceConfig/0/GenEthernet-1.1.dbc` will appear

16. Mapped channels are inside the generated `GenEthernet-1.1.dbc`

17. Download configuration to Device via IxAdmin (will be installed with ACT)![](/docs/download_config.png)
<img src="docs/IxAdmin.png" width=50% height=50%>

>Switch to Linux OS and copy the created files

18.   Build c-coderdbc according to the instruction given in the [readme](https://github.com/AdriveLivingLab/c-coderdbc/blob/9af0fbd15538052d8887cb7610b9b6477e62b497/README.md).

19.   Create the folder you want to output the data in ./build, e.g. ``` example ```

20.   Parse original database file (already used in 7.) to c-coderdbc CLI

```./coderdbc -dbc ./example.dbc -out ./example -drvname example_gw```

21. Find the generated data in the folder you created, e.g. ```cd ./example/ros```

22.  Copy the generated publisher into

`./ros-node/src/FRos.cpp` in line `#269`

23. Copy the generated messages into

` ./ros-node/src/FRos.cpp` in line `#348`

24. Copy the content of the msg folder to ```./ros-node/msg/```
25. Copy and replace the CMakeLists.txt into ```./ros-node/```
26. Copy and replace the convert.h and decode.h into ```./ros-node/include/``` 

27. Build ros node with 

```catkin_make```

The CMakeList.txt will be automatically adopted to the messages, whilst generating the outcome of the c-dbcparser.

[[↑ to top ↑](#table-of-contents)]

---

## [Start ROS node](#start-ros-node)

After compilation the following steps are required to start the node: 
1. ```source devel/setup.bash```
2. ```roslaunch ixxat_gw ixxat_gw.launch```
   
Now the node will start and establish the connection to the device. The topics will be published as soon as they are received by the gateway. Take a deeper look into the launchfile. There, you can change the IP to match your configuration etc.

[[↑ to top ↑](#table-of-contents)]

---

# [Known issues](#known-issues)

> **_ISSUE #1_:**  FlexRay allows for PDUs, and as such also its signals, to appear in more than one frame. This way, a specific PDU might be updated more frequently by, e.g. publishing it in two separate frames. Thus, with the analysis of only one of the two frames, there is data lost and the observed updaterate of the signals is halved. 

> **_SOLUTION #1.1_:** 
- Observe all frames that include this PDU, and with that the signal, and fuse them together in post-processing.
- Note: We are currently working on supporting the wildcard mapping feature of the device. With this, we want to be able to decode all FlexRay frames and publish the contained PDU's as ROS topics. Stay tuned for that.
> **_SOLUTION #1.2_:** 
- Change the configuration to map PDUs to GenEthernet, as such the signals will be updated by all the frames the PDU happens to be included.

[[↑ to top ↑](#table-of-contents)]


# [License](#license)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

# [Contact](#contact)

<div style="width: 50%; float:left" align="center">
  <a href="https://www.instagram.com/fza_hskempten/" style="text-decoration:none;"> &nbsp; <img src = "docs/instagram_ifm.svg" alt="instagram IFM" width=35px/></li> 
  <a href="https://www.youtube.com/@ifm8936" style="text-decoration:none;"> &nbsp;<img src = "docs/youtube_ifm.svg" alt="youtube IFM" width=35px/>
  <a href="https://de.linkedin.com/company/institute-for-driver-assistace-and-connected-mobility" style="text-decoration:none;"> &nbsp; <img src = "docs/linkedin_ifm.svg" alt="linkedin IFM" width=35px/>
  <a href="mailto:livinglab.info@hs-kempten.de" style="text-decoration:none;"> &nbsp; <img src = "docs/mail_ifm.svg" alt="email IFM" width=35px/>
  <br/>
  <a href="#" style="text-decoration:none;"> <img src = "docs/Kombi_Logo_farbig_freigestellt.png" alt="website FZA" width=200px/>
</div>

<div style="width: 50%; float:right" align="center">
  <a href="https://www.instagram.com/hmsnetworks/" style="text-decoration:none;"> &nbsp; <img src = "docs/instagram_hms.svg" alt="instagram HMS" width=35px/></li> 
  <a href="https://www.youtube.com/c/IxxatbyHMSNetworks" style="text-decoration:none;"> &nbsp;<img src = "docs/youtube_hms.svg" alt="youtube HMS" width=35px/>
  <a href="https://www.linkedin.com/company/hmsnetworks" style="text-decoration:none;"> &nbsp; <img src = "docs/linkedin_hms.svg" alt="linkedin HMS" width=35px/>
  <a href="mailto:info@hms-networks.de" style="text-decoration:none;"> &nbsp; <img src = "docs/mail_hms.svg" alt="email HMS" width=35px/>
  <br/>
  </br>
  <a href="#" style="text-decoration:none;"> <img src = "docs/hms_logo.png" alt="website HMS" width=100px/>
</div> 

[ [↑ to top ↑](#table-of-contents) ]


# [Cite](#cite)

```BibTeX
@article{Schneider2024,
  title = {fROS: A Generic Fieldbus Framework for ROS},
  ISSN = {2379-8858},
  url = {http://dx.doi.org/10.1109/TIV.2024.3365849},
  DOI = {10.1109/tiv.2024.3365849},
  journal = {IEEE Transactions on Intelligent Vehicles},
  publisher = {Institute of Electrical and Electronics Engineers (IEEE)},
  author = {Schneider, Daniel and Kastner, Ludwig and Schick, Bernhard and Watzenig, Daniel},
  year = {2024},
  pages = {1–16}
}

[ [↑ to top ↑](#table-of-contents) ]
```
