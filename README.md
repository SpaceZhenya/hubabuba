<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>ИИ Напоминания с музыкой</title>
    <style>
        body { font-family: Arial; padding: 20px; }
        input, button { padding: 10px; margin: 5px 0; width: 100%; }
        ul { list-style: none; padding-left: 0; }
        li { padding: 5px 0; }
    </style>
</head>
<body>
    <h1>ИИ Напоминания с музыкой</h1>

    <!-- Ручной ввод -->
    <input type="text" id="reminderInput" placeholder="Например: завтра в 18:30 позвонить маме">
    <button onclick="addManualReminder()">Добавить напоминание</button>

    <!-- Голосовой ввод -->
    <button onclick="startListening()">Говори напоминание</button>

    <ul id="reminderList"></ul>

    <!-- Добавим аудио (можно заменить ссылку на свою мелодию) -->
    <audio id="reminderMusic" src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3"></audio>

    <script src="https://cdn.jsdelivr.net/npm/chrono-node/dist/chrono.min.js"></script>
    <script>
        const reminders = [];
        const music = document.getElementById('reminderMusic');

        function speak(text) {
            const utterance = new SpeechSynthesisUtterance(text);
            utterance.lang = 'ru-RU';
            speechSynthesis.speak(utterance);
        }

        function addReminder(text) {
            const parsed = chrono.parseDate(text, new Date(), { forwardDate: true });
            if (!parsed) {
                speak("Не удалось распознать дату и время.");
                return;
            }
            reminders.push({ text, time: parsed });
            updateList();
            speak(`Напоминание добавлено на ${parsed.toLocaleString()}`);
        }

        function addManualReminder() {
            const input = document.getElementById('reminderInput');
            const text = input.value;
            if (!text) return;
            addReminder(text);
            input.value = '';
        }

        function updateList() {
            const list = document.getElementById('reminderList');
            list.innerHTML = '';
            reminders.forEach((r) => {
                const li = document.createElement('li');
                li.textContent = `${r.text} — ${r.time.toLocaleString()}`;
                list.appendChild(li);
            });
        }

        function checkReminders() {
            const now = new Date();
            for (let i = reminders.length - 1; i >= 0; i--) {
                if (now >= reminders[i].time) {
                    // Сначала проигрываем музыку
                    music.play().then(() => {
                        // Через 2 секунды озвучиваем текст (можно увеличить время)
                        setTimeout(() => speak(`Напоминание: ${reminders[i].text}`), 2000);
                    }).catch(() => {
                        // Если музыка не проигралась, просто озвучиваем
                        speak(`Напоминание: ${reminders[i].text}`);
                    });

                    reminders.splice(i, 1);
                    updateList();
                }
            }
        }
        setInterval(checkReminders, 10000);

        function startListening() {
            const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
            if (!SpeechRecognition) {
                alert("Ваш браузер не поддерживает голосовой ввод");
                return;
            }
            const recognition = new SpeechRecognition();
            recognition.lang = 'ru-RU';
            recognition.start();

            recognition.onresult = (event) => {
                const text = event.results[0][0].transcript;
                addReminder(text);
            };
            recognition.onerror = () => speak("Ошибка распознавания речи.");
        }

        if (Notification.permission !== "granted") {
            Notification.requestPermission();
        }
    </script>
</body>
</html>
