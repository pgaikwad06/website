# Sequence Diagram for whisper_online.py

```mermaid
sequenceDiagram
    participant User
    participant WhisperOnline
    participant OnlineProcessor

    User ->> WhisperOnline: append audio to audio_buffer
    WhisperOnline ->> WhisperOnline: np.append(self.audio_buffer, audio)

    alt res is not None
        WhisperOnline ->> WhisperOnline: frame = list(res.values())[0]
        
        alt 'start' in res and 'end' not in res
            WhisperOnline ->> WhisperOnline: self.status = 'voice'
            WhisperOnline ->> WhisperOnline: send_audio = self.audio_buffer[frame-self.buffer_offset:]
            WhisperOnline ->> OnlineProcessor: self.online.init(offset=frame/self.SAMPLING_RATE)
            WhisperOnline ->> OnlineProcessor: self.online.insert_audio_chunk(send_audio)
            WhisperOnline ->> WhisperOnline: self.current_online_chunk_buffer_size += len(send_audio)
            WhisperOnline ->> WhisperOnline: self.clear_buffer()
        
        else 'end' in res and 'start' not in res
            WhisperOnline ->> WhisperOnline: self.status = 'nonvoice'
            WhisperOnline ->> WhisperOnline: send_audio = self.audio_buffer[:frame-self.buffer_offset]
            WhisperOnline ->> OnlineProcessor: self.online.insert_audio_chunk(send_audio)
            WhisperOnline ->> WhisperOnline: self.current_online_chunk_buffer_size += len(send_audio)
            WhisperOnline ->> WhisperOnline: self.is_currently_final = True
            WhisperOnline ->> WhisperOnline: self.clear_buffer()
        
        else
            WhisperOnline ->> WhisperOnline: raise NotImplemented("both start and end of voice in one chunk!!!")
    
    else res is None
        alt self.status == 'voice'
            WhisperOnline ->> OnlineProcessor: self.online.insert_audio_chunk(self.audio_buffer)
            WhisperOnline ->> WhisperOnline: self.current_online_chunk_buffer_size += len(self.audio_buffer)
            WhisperOnline ->> WhisperOnline: self.clear_buffer()
