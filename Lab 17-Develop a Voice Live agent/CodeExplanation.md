# Azure VoiceLive Voice Agent – Detailed Code Explanation

This document provides a **complete, function-by-function explanation** of the given Azure VoiceLive voice agent code. Each section includes:
- Explanation of logic
- Key constructs used
- A summary table of parameters and constructs

---

# 1. `main()`

## What it does
This is the **entry point of the program**. It:
1. Clears console
2. Loads environment variables
3. Reads Azure configuration (endpoint, agent, project)
4. Creates credentials
5. Instantiates `VoiceAssistant`
6. Runs it asynchronously

## Key Flow
```
load_dotenv()
endpoint = os.environ.get(...)
```
→ Reads config from `.env`

```
credential = AzureCliCredential()
```
→ Uses Azure CLI login for auth

```
assistant = VoiceAssistant(...)
asyncio.run(assistant.start())
```
→ Starts async voice assistant loop

## Function Summary Table

| Element Type | Name | Purpose |
|-------------|------|--------|
| Function | main() | Program entry point |
| Library call | os.system() | Clears terminal |
| Function | load_dotenv() | Loads environment variables |
| Variable | endpoint, agent_name, project_name | Config values |
| Class init | AzureCliCredential() | Authentication |
| Class init | VoiceAssistant(...) | Creates assistant |
| Async runner | asyncio.run() | Runs async function |
| Exception | KeyboardInterrupt | Handles Ctrl+C |
| Exception | Exception | General error handling |

---

# 2. `VoiceAssistant.__init__()`

## What it does
Initializes the assistant with:
- Azure endpoint
- Credentials
- Agent configuration

```
self.agent_config = {
    "agent_name": agent_name,
    "project_name": project_name
}
```

## Function Summary Table

| Element Type | Name | Purpose |
|-------------|------|--------|
| Constructor | __init__ | Initializes class |
| Parameter | endpoint | Azure endpoint |
| Parameter | credential | Auth object |
| Parameter | agent_name | Agent ID |
| Parameter | project_name | Project |
| Variable | self.agent_config | Agent metadata dict |

---

# 3. `VoiceAssistant.start()`

## What it does
Core orchestration method:
1. Connect to Azure VoiceLive
2. Create AudioProcessor
3. Setup session
4. Start playback
5. Process events

## Function Summary Table

| Element Type | Name | Purpose |
|-------------|------|--------|
| Async function | start() | Main async orchestrator |
| Context manager | connect() | Opens Azure connection |
| Parameter | endpoint | Azure URL |
| Parameter | credential | Auth |
| Parameter | agent_config | Agent info |
| Class init | AudioProcessor() | Handles audio |
| Method call | setup_session() | Config session |
| Method call | start_playback() | Begin speaker output |
| Method call | process_events() | Event loop |
| Cleanup | shutdown() | Close audio |

---

# 4. `setup_session()`

## What it does
Defines behavior:
- Modalities (text + audio)
- Audio formats
- Voice activity detection
- Noise reduction

## Function Summary Table

| Element Type | Name | Purpose |
|-------------|------|--------|
| Async function | setup_session() | Configures session |
| Class | RequestSession | Session config object |
| Parameter | modalities | TEXT + AUDIO |
| Parameter | InputAudioFormat.PCM16 | Input format |
| Parameter | OutputAudioFormat.PCM16 | Output format |
| Class | AzureSemanticVadMultilingual | Speech detection |
| Class | AudioEchoCancellation | Echo removal |
| Class | AudioNoiseReduction | Noise suppression |
| Method | connection.session.update() | Applies config |

---

# 5. `process_events()`

## What it does
Continuously listens for server events.

```
async for event in self.connection:
    await self.handle_event(event)
```

## Function Summary Table

| Element Type | Name | Purpose |
|-------------|------|--------|
| Async function | process_events() | Event listener loop |
| Loop | async for | Stream events |
| Method call | handle_event() | Process each event |

---

# 6. `handle_event()`

## What it does
Handles different event types from VoiceLive.

Key events include:
- SESSION_UPDATED
- Transcription completed
- Agent response
- Audio streaming
- Errors

## Function Summary Table

| Element Type | Name | Purpose |
|-------------|------|--------|
| Async function | handle_event() | Event dispatcher |
| Enum | ServerEventType | Event types |
| Method | start_capture() | Begin mic input |
| Method | queue_audio() | Add audio chunk |
| Method | clear_playback_queue() | Stop playback |
| Property | event.transcript | Speech text |
| Property | event.delta | Audio chunk |
| Property | event.error.message | Error info |

---

# 7. `AudioProcessor.__init__()`

## What it does
Initializes audio system:
- PyAudio
- Format settings
- Streams
- Queue

## Function Summary Table

| Element Type | Name | Purpose |
|-------------|------|--------|
| Constructor | __init__() | Initialize audio processor |
| Class | pyaudio.PyAudio() | Audio engine |
| Variable | format | PCM16 |
| Variable | channels | Mono |
| Variable | rate | 24kHz |
| Variable | chunk_size | Buffer size |
| Class | queue.Queue() | Audio buffer |

---

# 8. `start_capture()`

## What it does
Starts microphone streaming:
- Capture audio
- Encode to Base64
- Send to Azure

## Function Summary Table

| Element Type | Name | Purpose |
|-------------|------|--------|
| Method | start_capture() | Starts mic |
| Callback | capture_callback | Handles audio frames |
| Library | base64.b64encode() | Encode audio |
| Function | asyncio.run_coroutine_threadsafe() | Send async |
| Stream | audio.open() | Open input stream |
| Parameter | frames_per_buffer | Chunk size |

---

# 9. `start_playback()`

## What it does
Plays audio from agent.

## Function Summary Table

| Element Type | Name | Purpose |
|-------------|------|--------|
| Method | start_playback() | Starts speakers |
| Callback | playback_callback | Outputs audio |
| Queue | playback_queue | Stores audio chunks |
| Stream | audio.open() | Output stream |
| Function | get_nowait() | Fetch audio |

---

# 10. `queue_audio()`

## What it does
Adds audio chunks to queue.

## Function Summary Table

| Element Type | Name | Purpose |
|-------------|------|--------|
| Method | queue_audio() | Add audio to queue |
| Queue method | put() | Insert audio |

---

# 11. `clear_playback_queue()`

## What it does
Clears queue when interrupted.

## Function Summary Table

| Element Type | Name | Purpose |
|-------------|------|--------|
| Method | clear_playback_queue() | Clear queue |
| Loop | while not empty() | Drain queue |
| Queue method | get_nowait() | Remove items |

---

# 12. `shutdown()`

## What it does
Stops and cleans audio resources.

## Function Summary Table

| Element Type | Name | Purpose |
|-------------|------|--------|
| Method | shutdown() | Cleanup resources |
| Method | stop_stream() | Stop mic/speaker |
| Method | close() | Close streams |
| Queue | put(None) | Signal end |
| Method | terminate() | Destroy PyAudio |

---

# Final Flow

```
main()
 ↓
VoiceAssistant.start()
 ↓
connect to Azure VoiceLive
 ↓
setup_session()
 ↓
start_playback()
 ↓
process_events()
 ↓
handle_event()
 ↓
AudioProcessor handles mic + speaker
```

---

# Key Concepts

- Event-driven architecture
- Async programming
- Real-time audio streaming
- Separation of concerns
