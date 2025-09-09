
Create a .env in the same folder as main.py. Fill with your values.
env

# Twilio (rotate your token!)
TWILIO_SID=ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_AUTH=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_CALLER_ID=+1234567890
# Publicly reachable URL Twilio should call (ngrok/cloud URL)
TWILIO_CALLBACK_URL=https://your-public-url

# LLM
USE_LOCAL_LLM=1
GGUF_MODEL_PATH=/models/model-q8_0.gguf
LLM_CTX=2048
LLM_THREADS=8
LLM_N_GPU_LAYERS=-1

# Gen knobs
AI_MAX_NEW_TOKENS=250
AI_TEMPERATURE=0.0
AI_TOP_P=1.0
AI_REP_PENALTY=1.0
AI_MAX_CHARS=220

# If using llama.cpp server instead of in-process:
# USE_LOCAL_LLM=0
# AI_SERVER_URL=http://127.0.0.1:8080
# AI_API_KEY=
Install deps (choose the CUDA wheel that matches your box)
Check CUDA version:
nvidia-smi (note the CUDA Version)
Install:

CUDA 12.4:
pip install "llama-cpp-python-cu124>=0.2.90"
CUDA 12.1:
pip install "llama-cpp-python-cu121>=0.2.90"
CPU-only:
pip install "llama-cpp-python>=0.2.90"
Plus app dependencies:

pip install fastapi uvicorn twilio python-dotenv httpx
Or as a single sequence (example for CUDA 12.1):

Bash

pip install "llama-cpp-python-cu121>=0.2.90" fastapi uvicorn twilio python-dotenv httpx
Run command
Assuming your model is at /models/model-q8_0.gguf and .env is in the same directory:
Bash

export PYTHONUNBUFFERED=1
uvicorn main:app --host 0.0.0.0 --port 8000
Minimal test before Twilio
POST to your endpoint to simulate Twilio:
Bash

curl -X POST "http://127.0.0.1:8000/gather-speech" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  --data-urlencode "SpeechResult=How can you help a tiles company?"
Check logs for something like “[x.xx s] AI generation”. If it’s slow, reduce AI_MAX_NEW_TOKENS to 24–64 and verify GPU offload (nvidia-smi).
Optional: Run as llama.cpp server instead of in-process
If you’d rather keep FastAPI lighter and talk to llama.cpp over HTTP:

Build/run the server (from your llama.cpp build):
Bash

./build/bin/llama-server -m /models/model-q8_0.gguf -c 2048 -ngl -1 -t 8
Set in .env:
USE_LOCAL_LLM=0
AI_SERVER_URL=http://127.0.0.1:8080
Restart your FastAPI app. The same routes will now call the HTTP server.

That’s it. This is the final updated code and the commands to run it on your Vast.ai GPU instance using your merged q8_0 GGUF.
