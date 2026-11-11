# Lab 3: Deploying a 5G Standalone Network Using OpenAirInterface

## 1. Goal

In the previous labs, you analyzed 5G signaling messages and network performance using provided captures and measurements.

In this lab, you will deploy your own 5G Standalone (SA) network using OpenAirInterface (OAI), verify the complete connection, investigate RFsimulator communication, perform active network measurements, and troubleshoot a controlled connection failure.

Unlike the previous labs, you will collect measurements directly from your own running 5G network.

**Learning Objectives**

After completing this lab, you should be able to:

1. Deploy an OAI gNB, nrUE, and 5G Core Network.
2. Identify the main stages of 5G SA connection establishment.
3. Verify communication between gNB and nrUE through RFsimulator.
4. Measure user-plane connectivity under different packet sizes.
5. Diagnose connection failures using OAI and 5GC logs.
6. Explain the limitations of a software-simulated 5G network.

## 2. Experimental Environment

Use the *OAI v2.4.0 Build Guide* to prepare the environment before class.

| Component | Configuration |
|---|---|
| Ubuntu | 24.04 |
| OAI RAN | v2.4.0 |
| 5G Core | OAI CN5G |
| gNB | nr-softmodem |
| UE | nr-uesoftmodem |
| Radio interface | RFsimulator |
| Band | n78 |
| Subcarrier spacing | 30 kHz |
| Resource blocks | 106 |
| UE IMSI | 001010000000001 |
| External DN | 192.168.70.135 |

The experiment uses one Ubuntu machine without physical RF hardware.

## 3. Task 1 — Deploy and Verify the Complete 5G SA Network

Start the OAI CN5G:

```bash
cd ~/oai-cn5g
docker compose up -d
docker compose ps
```

Start the gNB and nrUE using the commands in the Build Guide.

Observe their terminal outputs and locate evidence of the following stages.

| Stage | Successful? | Evidence from logs |
|---|---|---|
| CN5G initialization | Successful | <img width="668" height="379" alt="image" src="https://github.com/user-attachments/assets/69149e85-7821-442d-bd5a-0ff2ebb36360" />
 |
| gNB NG Setup | Successful | <img width="959" height="475" alt="image" src="https://github.com/user-attachments/assets/b206839d-5d64-4e99-a5c9-67d30e6824f6" />
 |
| UE synchronization | Succesful | <img width="664" height="375" alt="image" src="https://github.com/user-attachments/assets/1a146a24-0db6-43e2-a6d3-8755404807ec" />
 |
| RRC connection establishment | Successful | <img width="959" height="505" alt="image" src="https://github.com/user-attachments/assets/b37073b7-9856-408d-b92a-dcb92317e82c" />
 |
| UE Registration | Successful | <img width="959" height="500" alt="image" src="https://github.com/user-attachments/assets/2bb3dab5-aee4-46a0-a73e-454528c8ca1d" />
 |
| PDU Session Establishment | Successful | <img width="959" height="507" alt="image" src="https://github.com/user-attachments/assets/65a124bd-2360-4433-9e19-0debf2fb0bef" />
 |
| UE tunnel creation | Successful | <img width="670" height="345" alt="image" src="https://github.com/user-attachments/assets/090d314f-dc74-4897-b76c-a5574c72b426" />
 |

Take screenshots showing the gNB and UE successfully connected.

**Questions:**

1. Does a successful NG Setup guarantee that a UE can register? Explain. - No, NG setup only establishes the NG control plane connection between gNB and AMF, while UE registration requires successful authentication, security establishment and NAS signaling 
2. Why can the UE establish an RRC connection but still fail Registration? - RRC connection establishment and UE registration occur at different protocol layers 
3. What additional procedure is required after Registration before user data can be exchanged? - PDU Session Establishment 

## 4. Task 2 — Investigate RFsimulator Communication

OAI RFsimulator replaces the physical RF interface with software-based communication.

The gNB and nrUE exchange IQ samples through a TCP connection.

Check the running processes:

```bash
pgrep -a nr-softmodem
pgrep -a nr-uesoftmodem
```

Inspect the TCP connections:

```bash
sudo ss -tnp | grep 4043
```

If necessary, inspect the listener:

```bash
sudo ss -ltnp | grep 4043
```

The default RFsimulator port in this experiment is 4043.

Complete the table:

| Item | Observed Value |
|---|---|
| gNB process ID | 9675 |
| nrUE process ID | 12990 |
| RFsimulator transport protocol | TCP |
| RFsimulator port | 4043 |
| TCP connection state | ESTAB |
| Connected process(es) | nr-softmodem (gNB, PID 9675) , nr-uesoftmodem (nrUE, PID 12990) |

Take one screenshot showing the TCP connection. - <img width="864" height="142" alt="image" src="https://github.com/user-attachments/assets/244cfb78-21aa-4e2e-8beb-1af0138d9efd" />


**Questions:**

1. Why does RFsimulator use a TCP connection instead of a physical antenna? - it helps to provide reliable ordered delivery of simulated radio samples between the gNB and nrUE
2. Are the TCP packets exchanged by RFsimulator the same as the UE's application-layer IP packets? - No, they are different
3. Does using RFsimulator mean that the UE and gNB are performing no physical-layer processing? Explain. - No, RFsimulator does not eliminate the physical-layer processing, UE and gNB still performs the physical layer operations of modulation, demodulation, channel coding, decoding, synchronization and signaling, it only replaces the physical hardware

## 5. Task 3 — Verify End-to-End Connectivity

Check the UE tunnel interface:

```bash
ip -4 addr show oaitun_ue1
```

Test connectivity to the External DN:

```bash
ping -I oaitun_ue1 -c 10 192.168.70.135
```

Record:

| Measurement | Observed Value |
|---|---|
| UE IP address | 10.0.0.2/24 |
| Destination IP | 192.168.70.135 |
| Packets transmitted | 10 |
| Packets received | 10 |
| Packet loss (%) | 0 |
| Minimum RTT (ms) | 23.668 |
| Average RTT (ms) | 27.780 |
| Maximum RTT (ms) | 30.966 |

Take a screenshot of the Ping result.
<img width="791" height="254" alt="image" src="https://github.com/user-attachments/assets/2fa94720-0c0a-45c4-bb21-92bd844a805f" />


**Questions:**

1. Why must Ping use the `oaitun_ue1` interface in this experiment? - it allows the experiment to specifically test for connectivity through the OAI 5G network
2. Does successful Registration guarantee successful user-plane connectivity? - No, it does not
3. What does a successful Ping prove, and what does it not prove about the simulated 5G system? - it proves that end to end IP connectivity is established between the UE and the destination but does not prove that the 5G system accurately represents a real-world network performance

## 6. Task 4 — Investigate the Effect of Packet Size

You will now investigate whether changing the ICMP packet size affects the observed RTT.

Perform three experiments using the same established 5G connection.

**Scenario A — Small Packets**

```bash
ping -I oaitun_ue1 -s 56 -c 30 -i 0.2 192.168.70.135
```

**Scenario B — Medium Packets**

```bash
ping -I oaitun_ue1 -s 512 -c 30 -i 0.2 192.168.70.135
```

**Scenario C — Large Packets**

```bash
ping -I oaitun_ue1 -s 1200 -c 30 -i 0.2 192.168.70.135
```

Note: `-s` specifies ICMP payload size, not the complete IP packet size.

Complete the table:

| Scenario | ICMP Payload | Avg. RTT (ms) | Max. RTT (ms) | Packet Loss (%) |
|---|---:|---:|---:|---:|
| A | 56 bytes | 24.093 | 30.126 | 0 |
| B | 512 bytes | 28.732 | 34.950 | 0 |
| C | 1200 bytes | 35.188 | 61.250 | 0 |

Calculate the RTT difference:

```text
RTT Difference = Average RTT(C) - Average RTT(A)
```
11.095

**Questions:**

1. Which scenario has the highest average RTT? - Scenario C
2. Does increasing packet size always increase RTT in your measurements? - Yes
3. Why might a larger packet require more transmission or processing resources? - larger packets contain more data, which would need more transmission and processing resources 
4. What factors other than packet size could affect RTT in a software-based 5G network? - RTT can be affected by CPU utilization, memory availability, operating system scheduling, network congestion and virtualization overhead
5. Can these results be interpreted as real over-the-air 5G latency? Explain. - No

Do not assume that RTT must increase with packet size. Use your measured values to support your explanation.

## 7. Task 5 — Diagnose a Controlled UE Registration Failure

In this task, you will deliberately introduce one configuration error and investigate why the UE cannot complete registration.

**Important:** Save the original successful UE command before starting. Do not modify the CN5G subscriber database.

### Step 1 — Stop the UE

Press `Ctrl+C` in the nrUE terminal. Keep the CN5G and gNB running.

### Step 2 — Launch the UE with an Incorrect IMSI

Change only the IMSI:

```bash
sudo ./nr-uesoftmodem \
  -r 106 \
  --numerology 1 \
  --band 78 \
  -C 3619200000 \
  --uicc0.imsi 001010000000099 \
  --rfsim \
  --rfsimulator.serveraddr 127.0.0.1
```

The IMSI `001010000000099` is not provisioned in the standard classroom subscriber configuration. Verify this assumption against the instructor's database before class.

Observe the UE and AMF logs.

```bash
cd ~/oai-cn5g
docker compose logs --since=2m oai-amf
```

Allow time for registration attempts, then stop the UE.

Complete the table:

| Item | Observation |
|---|---|
| Modified parameter | IMSI |
| UE synchronized with gNB? | Yes |
| RRC connection established?  Yes| |
| Registration successful? | No |
| Error or rejection evidence | <img width="628" height="314" alt="image" src="https://github.com/user-attachments/assets/ca1eb719-d71b-4b05-97f7-6955a79f3cff" />
 |
| Likely cause | "Received Registration reject message too short" |

If the expected registration rejection does not appear, report the actual logs instead of inventing a failure message.

### Step 3 — Restore the Original IMSI

Restart the nrUE using:

```text
--uicc0.imsi 001010000000001
```

Confirm that the UE successfully registers again and that Ping works. - <img width="954" height="143" alt="image" src="https://github.com/user-attachments/assets/055f77ff-aabf-467b-9ea0-8b160a4cb0ad" />


**Questions:**

1. Why can the UE potentially synchronize with the gNB even when the IMSI is incorrect? -UE can synchronize with the gNB because radio synchronization occurs at the physical layer and does not require IMSI authentication
2. Which network function is involved in UE authentication? - Authentication Server Function
3. Why does changing an IMSI affect Registration but not the RFsimulator TCP connection itself? - changing the IMSI affects registration because the IMSI identifies the subscriber and is used during authentication with the 5G Core Network
4. What evidence proves that restoring the correct configuration solved the problem? - the evidence is seen from the "Authentication successful by network" and "5GMM-Registered" messages

Submit evidence from both the failed and recovered connections.


## 8. Task 7 — Explain the Relationship Between Simulation and Real 5G

Your OAI gNB and nrUE perform 5G NR protocol processing even though they do not use physical antennas.

Complete the comparison:

| Feature | OAI RFsimulator | Physical 5G System |
|---|---|---|
| gNB and UE protocol stack | implemented in software | implemented in hardware and software |
| OFDM signal processing | performed using software | performed using physical systems |
| Physical antennas | none | needed for wireless transmissions |
| RF hardware impairments | not actually present, it is simulated | present |
| Wireless propagation effects | not actually present, it is modelled | present |
| 5G Core connectivity | done through simulation | supported through a 5G core |

**Questions:**

1. Which parts of the 5G system remain functional when physical RF hardware is replaced with RFsimulator? - the 5G protocol stack remains functional such as the PHY, MAC, RLC, PDCP, RRC and NAS layers. Physical-layer operations such as OFDM modulation, demodulation, channel coding and decoding are still performed in software.
2. Why is RFsimulator useful for testing the 5G protocol stack? - it allows testing without the need of expensive equipment 
3. Why might latency and throughput differ between RFsimulator and a real 5G deployment? - RFsimulator does not inherently reproduce real-world wireless conditions such as interference, fading, path loss, and mobility
4. What would need to change if you wanted to test a physical UE instead of OAI nrUE? - the simulated radio interface would need to be replaced with compatible RF hardware

## 9. Final Analysis

Answer each question in three to five sentences.

1. Describe the complete process from starting the OAI 5GC to successful user-plane connectivity. - OAI 5G Core Network is started by docker, the gNB starts and establishes an NG connection and the nrUE connects to the gNB through RFSimulator, the UE then undergoes authentication and registration with 5G Core
2. Explain the difference between an RFsimulator TCP connection and a 5G PDU Session. - RFsimulator TCP connection is used to transport simulated radio samples between the gNB and nrUE. While, a 5G PDU Session is a logical data connection established between the UE and the 5G data network through the 5G Core
3. Based on your measurements, how did packet size affect RTT and packet loss? - increasing packet size resulted in an increase in RTT, as larger packets require more transmission resources. However, packet loss remained at 0% from the measurements for all different packet sizes.
4. Describe the registration failure you introduced and explain how you diagnosed and recovered from it. - a controlled registration failure was introduced by changing the UE IMSI from 001010000000001 to 001010000000099. The failure was investigated by examining the AMF logs which showed that the modified UE remained in the 5GMM-DEREGISTERED state. To recover, the nrUE was stopped and restarted using the original IMSI 001010000000001
5. Identify two limitations of this simulated environment and explain how they might affect experimental results. - the absence of realistic wireless propagation conditions, it may result it the accuracy of the signal quality in a real-world scenario. dependence on software and host computing resources, this could result in processing delays, as it is dependent on the computer processing power

## 10. Submission Checklist

Submit one Markdown or PDF report containing:

- Completed 5G SA connection verification table.
- gNB and nrUE connection screenshots.
- RFsimulator TCP connection evidence.
- Baseline Ping measurement.
- Three packet-size comparison results.
- RTT difference calculation.
- Controlled failure and recovery evidence.
- RFsimulator versus physical 5G comparison.
- Answers to the final analysis questions.

All measurements and log evidence must come from your own experimental environment.

## 11. Grading Rubric

| Item | Points |
|---|---:|
| Complete 5G SA deployment | 15 |
| RFsimulator TCP connection analysis | 15 |
| End-to-end user-plane verification | 15 |
| Packet-size experiment and analysis | 20 |
| Controlled failure and recovery | 20 |
| Simulation limitations and final analysis | 15 |
| **Total** | **100** |

Vote TOP3：https://forms.gle/Mvza68DFBzPcZXfW6
