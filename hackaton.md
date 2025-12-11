Pre-Hackathon Tech Checklist (PM & Owners)

To be completed 24 hours BEFORE the event.

General

[ ] Pizza/Snacks ordered? (Crucial for 4-hour intensity).

[ ] WiFi access confirmed for all personal laptops.

[ ] All participants have GitHub access to TMC-Italia org.

☁️ Cloud Project Prep

[ ] Hardware: Are the legacy PCs plugged in and on the network?
[ ] OS: Is there at least one machine with a fresh Ubuntu Server install ready to be the "test subject"?
[ ] Secrets: Are the Tailscale Auth Keys generated and saved in a .env file or password manager?

🧠 AI Project Prep

[ ] Data: Do we have a folder with 10-20 anonymized dummy CVs? (Do NOT use real sensitive data for the hackathon to avoid GDPR headaches during dev).
[ ] Models: Has someone pre-downloaded the sentence-transformers model weights? (Downloading 2GB+ during the hackathon kills WiFi).
[ ] Docker: Does the docker-compose.yml for ChromaDB/Ollama work on Alessandro's machine?

📍 TMC Places Prep

[ ] Keys: Create a mock_data.json file so devs don't need real Microsoft Graph API tokens to start working.
[ ] Design: Do we have the SVG or image file of the office floorplan? (Devs cannot draw the map during the 4 hours).


4-Hour Hackathon Execution Plan: TMC Innovation Projects

1. Logistics & Setup

Duration: 4 Hours

Participants: 7-8 People

Goal: Ship one "Vertical Slice" or "Core Component" per project. No "In Progress" code at the end—everything must run or demo.

2. Team Structure & Roles (Total: 8 Pax)

To manage PRs efficiently, we will split into two "Review Squads." The PR Reviewers/Owners are responsible for unblocking devs and merging code immediately.

Squad A: Infrastructure & Cloud (3 People)

Owner/Reviewer: Silvio (Group Leader/PM)

Developers:

Marco (Hardware/OS focus)

Flavio (Network/Storage focus)

Project: Hosting and Cloud

Squad B: Software & AI (5 People)

Owner/Reviewer: Alessandro (Tech Lead/Architect)

Developers (AI Team):

Matilde (Data/Curricular focus)

Dev 1 (TBD)

Developers (Places Team):

Dev 2 (Frontend focus)

Dev 3 (Backend focus)

3. The 4-Hour Schedule (The "Sprint")

Time

Phase

Activity

0:00 - 0:15

Kick-off

PM (Silvio) sets the goal. Everyone clones latest repos. Environment check (Do npm install / pip install work?).

0:15 - 1:45

Sprint Block 1

"The Ugly Implementation". Code the logic. Don't worry about perfect clean code yet. Get the feature working locally.

1:45 - 2:00

Sync & Coffee

Stand-up. Blockers raised. If a feature isn't 50% done, cut the scope now.

2:00 - 3:15

Sprint Block 2

"Integration & Refinement". Connect Frontend to Backend, or Script to Server. Prepare the PR.

3:15 - 3:45

The Merge

Code Freeze. Reviewers (Silvio/Alessandro) review PRs. CI/CD checks. Merge to dev/main.

3:45 - 4:00

Demo

5 mins per team. Show it running. No slides, only live code.

4. Tactical Objectives (The "Must-Dos")

☁️ Project 1: Cloud / On-Prem Infra

Context: Repurposing legacy PCs.
Hackathon Goal: "One-Click Node Setup"

Why: Installing OS and config manually takes too long. We need automation.

Tasks:

Marco: Finalize the tailscale and Network Config script. Ensure the node joins the mesh automatically upon run.

Flavio: Write the Master Setup Script (Bash/Ansible) that calls Docker installation, Portainer agent, and Storage mount.

Silvio (Reviewer): Test the script on a fresh node/VM to verify it works "headless."

🧠 Project 2: AI (HR CV Screening)

Context: Local RAG, GDPR compliant, no internet API.
Hackathon Goal: "The Ingestion Pipeline"

Why: We can't do RAG without clean data. The parsing is the bottleneck.

Tasks:

Matilde: Build the PDF Cleaner. Script using PyMuPDF or unstructured to strip headers/footers/images and output clean JSON/Text.

Dev 1: Build the Embedding Generator. Take Matilde's text, run sentence-transformers (Italian model), and insert vectors into ChromaDB (running in Docker).

Alessandro (Reviewer): Ensure the docker-compose for the Vector DB is stable and Python environments match.

📍 Project 3: TMC Places

Context: Booking visualization, Graph API.
Hackathon Goal: "The Map Connection"

Why: We have a backend and frontend, but do they talk?

Tasks:

Dev 2 (Frontend): Create the Office Map Component (SVG/Canvas). Hardcode the rooms, but make them change color based on a prop (status='occupied').

Dev 3 (Backend): Create a Mock Endpoint in FastAPI that mimics the Microsoft Graph response (so we don't spend 4 hours debugging OAuth). Serve a JSON of room statuses.

Integration: Fetch the Mock JSON in Frontend to update the Map colors.

5. Definition of Done (DoD)

Cloud: A script exists. When run on a fresh Ubuntu Server, it installs Docker, connects Tailscale, and reports "Ready."

AI: A Python script exists. You point it at a folder of 5 PDFs, and it results in a populated Vector Database (verified by a simple query print).

Places: You can load localhost:3000, see a map of the office, and the rooms are colored red/green based on data coming from localhost:8000.
