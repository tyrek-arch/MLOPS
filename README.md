1. Pobranie i Przygotowanie Pliku Modelu
```bash
curl -LO https://huggingface.co/mozilla-ai/llamafile_0.10/resolve/main/Qwen3.5-0.8B-Q8_0.llamafile
sudo mkdir -p /opt/llamafile
sudo mv Qwen3.5-0.8B-Q8_0.llamafile /opt/llamafile/model.llamafile
sudo chmod +x /opt/llamafile/model.llamafile
```

2. Utworzenie Użytkownika Systemowego
```bash
sudo useradd --system --no-create-home llamafile
sudo chown -R llamafile:llamafile /opt/llamafile
```

3. Plik Jednostki Systemd
```ini
tyrek@Tyrek:~$ cat /etc/systemd/system/llamafile.service
[Unit]
Description=llamafile service
After=network-online.target
Wants=network-online.target

[Service]
User=llamafile
Environment="HOME=/tmp"
ExecStart=/bin/sh -c '/opt/llamafile/model.llamafile --server --host 192.168.0.71 --port 8080'
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

4. Uruchomienie i Aktywacja Usługi
```bash
sudo systemctl daemon-reload
sudo systemctl enable llamafile
sudo systemctl start llamafile
```

5. Weryfikacja Działania
Nasłuchiwanie na porcie (ss -tlnp)
```text
tyrek@Tyrek:~$ sudo ss -tlnp
State      Recv-Q     Send-Q         Local Address:Port          Peer Address:Port     Process                                                                         
LISTEN     0          128                  0.0.0.0:22                 0.0.0.0:*        users:(("sshd",pid=1934,fd=6))                                                 
LISTEN     0          128             192.168.0.71:8080               0.0.0.0:*        users:((".ape-1.10",pid=1925,fd=4))                                            
LISTEN     0          4096               127.0.0.1:631                0.0.0.0:*        users:(("cupsd",pid=1920,fd=7))                                                
LISTEN     0          128                     [::]:22                    [::]:*        users:(("sshd",pid=1934,fd=7))                                                 
LISTEN     0          4096                    [::1]:631                  [::]:*        users:(("cupsd",pid=1920,fd=6))
```

Status usługi i zużycie RAM (systemctl status)
```text
tyrek@Tyrek:~$ sudo systemctl status llamafile
● llamafile.service - llamafile service
     Loaded: loaded (/etc/systemd/system/llamafile.service; enabled; preset: enabled)
     Active: active (running) since Mon 2026-10-26 21:22:02 CET; 57min ago
 Invocation: 2347a9c555c54f68badc73b2275528f9
   Main PID: 1921 (sh)
      Tasks: 14 (limit: 8250)
     Memory: 1.9G (peak: 1.9G)
        CPU: 35.312s
     CGroup: /system.slice/llamafile.service
             ├─1921 /bin/sh -c "/opt/llamafile/model.llamafile --server --host 192.168.0.71 --port 8080"
             └─1925 /tmp/.ape-1.10 /opt/llamafile/model.llamafile --server --host 192.168.0.71 --port 8080
```

Zapytanie 
```bash
tyrek@Tyrek:~$ curl http://192.168.0.71:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "Qwen3.5-0.8B",
    "messages": [{"role": "user", "content": "siema"}]
  }'
{"choices":[{"finish_reason":"stop","index":0,"message":{"role":"assistant","content":"Słowa te są bardzo rzadkie, prawdopodobnie używane w kontekście:\n\n* **Materiału:** W języku polskim jest to słowo oznaczające \"materiały\" (np. \"przemyślanie materiałów\"). Jeśli chodzi o to, że masz pytania dotyczące materiałów, może to być pomocne!\n* **Języka angielskiego:** Może oznaczać *siema* jako wiersz lub tekst? W języku angielskim słowo ta jest bardzo rzadkie.\n\nJeśli chcesz powiedzieć coś więcej lub potrzebujesz pomocy w jakimś konkretnym pytaniu, podaj mi więcej szczegółów!"}}],"created":1793050153,"model":"/zip/Qwen3.5-0.8B-Q8_0.gguf","system_fingerprint":"b17894687250-0177dcc73","object":"chat.completion","usage":{"completion_tokens":141,"prompt_tokens":14,"total_tokens":155,"prompt_tokens_details":{"cached_tokens":10}},"id":"chatcmpl-0frXFKZ19ENJ3Q6ag06Khofy5Ud5nAv","timings":{"cache_n":10,"prompt_n":4,"prompt_ms":986.283,"prompt_per_tokentyrek@Tyrtyrek@Tyrtyrek@Tyrtyrek@Tyrtyrek@Tyrtyrek@Tyrtyrek@Tyrek:~$ S
```
