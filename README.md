# three
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>LOCUSCASE1 - Визуальный профиль университета</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; max-width: 800px; margin: 0 auto; padding: 20px; }
        .search-box { display: flex; gap: 10px; margin-bottom: 20px; }
        input { flex-grow: 1; padding: 10px; font-size: 16px; border: 1px solid #ccc; border-radius: 4px; }
        button { padding: 10px 20px; font-size: 16px; background-color: #007bff; color: white; border: none; border-radius: 4px; cursor: pointer; }
        button:hover { background-color: #0056b3; }
        #progress-container { display: none; margin-bottom: 20px; }
        .progress-bar { width: 100%; background-color: #f3f3f3; border-radius: 4px; overflow: hidden; }
        .progress-fill { height: 20px; background-color: #28a745; width: 0%; transition: width 0.3s; }
        #status-text { margin-top: 5px; font-size: 14px; color: #555; }
        .gallery { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 15px; }
        .card { border: 1px solid #ddd; border-radius: 8px; overflow: hidden; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        .card img { width: 100%; height: auto; display: block; }
        .card-info { padding: 10px; }
        .badge { display: inline-block; padding: 3px 8px; border-radius: 12px; font-size: 12px; background: #eee; margin-bottom: 5px; }
    </style>
</head>
<body>
    <h1>Поиск кампуса за 30 секунд</h1>
    
    <div class="search-box">
        <input type="text" id="uniInput" placeholder="Введите название университета (например, MIT)...">
        <button onclick="startSearch()">Анализировать</button>
    </div>

    <div id="progress-container">
        <div class="progress-bar">
            <div class="progress-fill" id="progress-fill"></div>
        </div>
        <div id="status-text">Инициализация...</div>
    </div>

    <div class="gallery" id="results"></div>

    <script>
        function startSearch() {
            const query = document.getElementById('uniInput').value;
            if (!query) return;

            const progressContainer = document.getElementById('progress-container');
            const progressFill = document.getElementById('progress-fill');
            const statusText = document.getElementById('status-text');
            const resultsDiv = document.getElementById('results');
            
            progressContainer.style.display = 'block';
            progressFill.style.width = '0%';
            resultsDiv.innerHTML = "";

            // Подключаемся к FastAPI через Server-Sent Events
            const eventSource = new EventSource(`http://localhost:8000/api/search?q=${encodeURIComponent(query)}`);
            
            eventSource.onmessage = function(event) {
                const data = JSON.parse(event.data);
                
                progressFill.style.width = `${data.progress}%`;
                statusText.innerText = data.status;
                
                if (data.progress === 100) {
                    eventSource.close();
                    
                    data.images.forEach(img => {
                        resultsDiv.innerHTML += `
                            <div class="card">
                                <img src="${img.url}" alt="${img.category}">
                                <div class="card-info">
                                    <span class="badge">${img.category}</span>
                                    <div style="font-size: 13px; margin-top: 5px;">
                                        <strong>Достоверность:</strong> ${img.trust_score}<br>
                                        <a href="#" style="color: #007bff; text-decoration: none;">Источник: ${img.source}</a>
                                    </div>
                                </div>
                            </div>
                        `;
                    });
                }
            };

            eventSource.onerror = function() {
                statusText.innerText = "Ошибка соединения с сервером";
                eventSource.close();
            };
        }
    </script>
</body>
</html>