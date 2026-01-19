# Intelligent Interruption Handling (for LiveKit Agents)

This fork implements an **Intelligent Interruption Logic Layer** that prevents the agent from stopping abruptly when the user provides "backchannel" feedback (e.g., "Yeah", "Uh-huh", "Okay").

## 🚀 The Feature

In standard Voice Agents, VAD (Voice Activity Detection) triggers immediately when sound is detected, stopping the agent. This creates a jarring experience if the user just says "Yeah" to confirm listening.

**Our Solution:**
1.  **Semantic Analysis**: We inspect the *content* of the user's speech using Real-time STT.
2.  **Ignored Words**: If the user's input consists entirely of "ignored words" (e.g., "yeah", "sure", "correct"), the agent **continues speaking**.
3.  **Command Prioritization**: If the input contains *any* other word (e.g., "Yeah wait"), the agent **interrupts immediately**.
4.  **VAD Override**: We disable "dumb" VAD interruption in favor of "smart" STT-based interruption.

## 🛠 Configuration

You can customize the list of ignored words using an environment variable.

| Variable | Description | Default |
|----------|-------------|---------|
| `LIVEKIT_IGNORE_WORDS` | Comma-separated list of words to ignore. | `yeah,ok,okay,hmm,uh-huh,right,sure` |

**Example:**
```bash
export LIVEKIT_IGNORE_WORDS="yeah,yep,uh-huh,continue"
```

## 🧪 How It Works (Architecture)

1.  **VAD Interruption Disabled**: The low-level `on_vad_inference_done` handler no longer triggers `interrupt()`.
2.  **STT Stream Handler**: Inside `_interrupt_by_audio_activity` (called when STT events arrive):
    *   We normalize the text (remove punctuation, lowercase).
    *   We match it against the `IGNORE_WORDS` list.
    *   **Match**: Log "Ignoring interruption" and do nothing.
    *   **No Match**: Trigger `interrupt()` (Stop audio, clear queue).

## ✅ Verification

You can verify the logic without running a full agent using the included test runner:

```bash
python run_logic_test.py
```

Expected Output:
```
Running Interruption Logic Tests...
[PASS] Ignore 'Yeah'
[PASS] Interrupt on 'Stop'
...
All tests passed!
```

## 📂 Key Files Modified

*   `livekit/agents/voice/interruption_logic.py`: Core logic for text analysis.
*   `livekit/agents/voice/agent_activity.py`: Integration into the event loop.
*   `tests/test_interruption_logic.py`: Unit tests.

---
(Original README follows below)
