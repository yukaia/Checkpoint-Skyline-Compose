# Instruction on enabling the OpenTelemetry agent on supported Gaia systems.

You can find the documentation for the minimum supported versions/jumbo hotfix takes of Gaia at the main Skyline SK article [sk178566](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk178566#Configuration) under the `Configuration` subsection D. 

| Major Version | Minimum JHF Take |
| ------------- | ---------------- |
| R81.20        | Take 8           |
| R81.10        | Take 79          |
| R81           | Take 77          |
| R80.40        | Take 190         |

If you have automatic updates enabled per [sk94508](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk94508) on your gateways/management servers then they should should automatically install/update the OpenTelemetry agent.

---

[payload-no-tls.json](payload-no-tls.json) is the configuration file to use if you didn't configure prometheus to listen via TLS. 

[payload-tls.json](payload-tls.json) is the configuration file to use if you did configure prometheus to listen via TLS. Checkpoint requires basic authentication if TLS is enabled.

There's a couple ways to enable and configure the OpenTelemetry agent in Gaia. The first method is locally and the second method is via the API starting with version 1.7 and up.

----

## Local Method
- first ssh into the box, drop into `expert` mode and download the respective configuration file.
  - If you want to you can download it directly from this repo using the following command. 
    - For the non-tls `curl_cli -kO "https://raw.githubusercontent.com/yukaia/Checkpoint-Skyline-Compose/main/checkpoint/payload-no-tls.json"`
    - For tls `curl_cli -kO "https://raw.githubusercontent.com/yukaia/Checkpoint-Skyline-Compose/main/checkpoint/payload-tls.json"`
      - alternatively you can create it using vi.
    - then edit the perspective file you downloaded.
      - replace the `<EXTERNAL_PROMETHEUS_IP_ADDRESS>` with either the IP address or hostname of your prometheus server. If you configured TLS then also modify the `<USERNAME` and `<PASSWORD>` sections as well as the public `<CERTIFICATE>` that Prometheus is providing.
      - from there we save the file and run one of the two commands.
        - No TLS Payload `/opt/CPotelcol/REST.py --set_open_telemetry "$(cat payload-no-tls.json)"`
        - TLS Payload `/opt/CPotelcol/REST.py --set_open_telemetry "$(cat payload-tls.json)"`
    - or you can do all the aforementioned work on your workstation and then copy/paste the payload text into the command and then run this in the expert shell on the gateway/management server in question.
     - `/opt/CPotelcol/REST.py --set_open_telemetry "<PayloadTextHere>"`

----

## API Method

First open the text editor or API tool of your choice and copy/past this API command into the workspace.


    POST {server}/v1.7/set-open-telemetry
    Content-Type: application/json
    X-chkp-sid: {{session}}
    {
        <PayloadHere>
    } 

Proceed to modify the `payload.json` file of your choice to match your environment and then replace the `<PayloadHere>` section in the above API call.

----

To disable the OpenTelemetry agent completely replace the `"enabled": true,` option in the payload to `"enabled": false,` and run the command.

----

Configuring Skyline for Maestro is a bit out of scope for this project as I don't have anything to test against at the moment, however `Configuration` subsection D also covers the Maestro/SP codebase. I've copied the instructions and pasted them below.

    - In a Maestro environment:

      - For the Maestro Hyperscale Orchestrator (MHO): You can run the script or run the Gaia Rest API. The script runs on the MHO and configures only the MHO.
      
      - For a Security Group: Run the script only on the Single Management Object (SMO). The SMO applies it to all the Security Group Members.
      
      - If there are issues with the script, try to download it from here and replace the current /opt/CPotelcol/REST.py file. 
      
      - The script uses gexec and g_cp2blades commands. Make sure they work correctly.