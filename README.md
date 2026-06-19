
<!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>होटल रूम मैनेजर</title>
    <style>
        :root {
            --available: #d4edda;
            --available-border: #c3e6cb;
            --available-text: #155724
            --occupied: #fff3cd;
            --occupied-border: #ffeeba;
            --occupied-text: #856404;
            --timeup: #f8d7da;
            --timeup-border: #f5c6cb;
            --timeup-text: #721c24;

 --bg: #f8f9fa;
            --text: #333;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg);
            color: var(--text);
            margin: 0;
            padding: 20px;
        }

        header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 20px;
            padding-bottom: 10px;
            border-bottom: 2px solid #ddd;
        }

        h1 { margin: 0; font-size: 28px; }
        
        button {
            padding: 10px 15px;
            cursor: pointer;
            border: none;
            border-radius: 5px;
            background-color: #007bff;
            color: white;
            font-weight: bold;
            font-size: 14px;
        }

        button:hover { background-color: #0056b3; }

        /* Red Buttons for Check-in and Check-out */
        .btn-red { background-color: #dc3545; width: 100%; margin-top: 10px; font-size: 16px; }
        .btn-red:hover { background-color: #c82333; }

        /* Grid Layout */
        .grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
            gap: 15px;
        }

        .room-card {
            border: 2px solid;
            border-radius: 8px;
            padding: 15px;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            box-shadow: 0 4px 6px rgba(0,0,0,0.05);
            transition: transform 0.2s;
        }

        .room-card:hover { transform: translateY(-3px); }
        .room-header { font-size: 24px; font-weight: bold; margin-bottom: 10px; }
        .status-badge { display: inline-block; padding: 5px 10px; border-radius: 15px; font-size: 13px; font-weight: bold; margin-bottom: 10px; }
        .timer-text { font-size: 15px; font-weight: bold; margin: 10px 0; color: #d32f2f; }
        .info-text { font-size: 14px; margin: 4px 0; }

        /* Status Colors */
        .status-available { background-color: var(--available); border-color: var(--available-border); color: var(--available-text); text-align: center; min-height: 150px; }
        .status-available .status-badge { background-color: var(--available-border); }
        
        .status-occupied { background-color: var(--occupied); border-color: var(--occupied-border); color: var(--occupied-text); }
        .status-occupied .status-badge { background-color: var(--occupied-border); }
        
        .status-timeup { background-color: var(--timeup); border-color: var(--timeup-border); color: var(--timeup-text); animation: blink 2s infinite; }
        .status-timeup .status-badge { background-color: var(--timeup-border); }

        @keyframes blink { 0% { box-shadow: 0 0 5px red; } 50% { box-shadow: 0 0 15px red; } 100% { box-shadow: 0 0 5px red; } }

        /* Modal */
        .modal-overlay {
            display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.5); justify-content: center; align-items: center; z-index: 1000;
        }
        .modal {
            background: white; padding: 25px; border-radius: 8px; width: 320px;
            box-shadow: 0 5px 15px rgba(0,0,0,0.3);
        }
        .modal h2 { margin-top: 0; }
        .form-group { margin-bottom: 15px; }
        .form-group label { display: block; margin-bottom: 5px; font-weight: bold; }
        .form-group input { width: 100%; padding: 8px; border: 1px solid #ccc; border-radius: 4px; box-sizing: border-box; font-size: 16px; }
        .modal-actions { display: flex; justify-content: space-between; margin-top: 20px; }
        .btn-cancel { background-color: #6c757d; }
        .btn-cancel:hover { background-color: #5a6268; }

        /* Logs Section */
        #logs-section { display: none; margin-top: 30px; background: white; padding: 20px; border-radius: 8px; border: 1px solid #ddd; }
        table { width: 100%; border-collapse: collapse; margin-top: 10px; }
        th, td { border: 1px solid #ddd; padding: 10px; text-align: left; font-size: 14px; }
        th { background-color: #f2f2f2; }
    </style>
</head>
<body>

    <header>
        <h1>होटल रूम मैनेजर</h1>
        <button onclick="toggleLogs()">पुराना रिकॉर्ड / लॉग देखें</button>
    </header>

    <div id="room-grid" class="grid">
        </div>

    <div id="logs-section">
        <h2>चेक-आउट का पुराना रिकॉर्ड</h2>
        <table>
            <thead>
                <tr>
                    <th>कमरा नंबर</th>
                    <th>चेक-इन (आने का समय)</th>
                    <th>चेक-आउट (जाने का समय)</th>
                    <th>कितने समय के लिए बुक था</th>
                </tr>
            </thead>
            <tbody id="logs-body">
                </tbody>
        </table>
    </div>

    <div id="checkin-modal" class="modal-overlay">
        <div class="modal">
            <h2 id="modal-room-title">कमरे में चेक-इन करें</h2>
            
            <div class="form-group">
                <label>प्रवेश का वर्तमान समय:</label>
                <div id="modal-current-time" style="font-size: 14px; color: #555;"></div>
            </div>

            <div class="form-group">
                <label for="input-days">कितने दिन के लिए:</label>
                <input type="number" id="input-days" min="0" value="1">
                <small style="color: gray; font-size: 12px; display: block; margin-top: 4px;">1 दिन = पूरे 24 घंटे</small>
            </div>

            <div class="form-group">
                <label for="input-hours">अतिरिक्त घंटे (अगर चाहिए):</label>
                <input type="number" id="input-hours" min="0" value="0">
                <small style="color: gray; font-size: 12px; display: block; margin-top: 4px;">कुछ घंटों के लिए या अतिरिक्त समय के लिए</small>
            </div>

            <div class="modal-actions">
                <button class="btn-cancel" onclick="closeModal()">रद्द करें</button>
                <button onclick="confirmCheckIn()">चेक-इन पक्का करें</button>
            </div>
        </div>
    </div>

    <script>
        // Core configuration
        const ROOM_NUMBERS = [1, 2, 3, 4, 5, 6, 7, 8, 9, 11, 12, 13, 14, 15, 16];
        
        // State management
        let rooms = JSON.parse(localStorage.getItem('hotel_rooms_state')) || {};
        let logs = JSON.parse(localStorage.getItem('hotel_logs_state')) || [];
        let selectedRoomForCheckin = null;

        // Initialize rooms if they don't exist in local storage
        if (Object.keys(rooms).length === 0) {
            ROOM_NUMBERS.forEach(num => {
                rooms[num] = {
                    status: 'available', // available, occupied, timeup
                    checkInTime: null,
                    endTime: null,
                    bookedDurationText: ''
                };
            });
            saveState();
        }

        // DOM Elements
        const gridElement = document.getElementById('room-grid');
        const modalOverlay = document.getElementById('checkin-modal');
        const logsSection = document.getElementById('logs-section');
        const logsBody = document.getElementById('logs-body');

        // Helper: Format Dates in Indian standard English format
        function formatDate(timestamp) {
            if (!timestamp) return 'N/A';
            const d = new Date(timestamp);
            return d.toLocaleString('en-IN'); 
        }

        // Save to LocalStorage
        function saveState() {
            localStorage.setItem('hotel_rooms_state', JSON.stringify(rooms));
            localStorage.setItem('hotel_logs_state', JSON.stringify(logs));
        }

        // Render the Grid
        function renderGrid() {
            gridElement.innerHTML = '';
            
            ROOM_NUMBERS.forEach(num => {
                const roomData = rooms[num];
                const card = document.createElement('div');
                
                if (roomData.status === 'available') {
                    card.className = 'room-card status-available';
                    card.innerHTML = `
                        <div>
                            <div class="room-header">कमरा ${num}</div>
                            <div class="status-badge">खाली है</div>
                        </div>
                        <button class="btn-red" onclick="openCheckInModal(${num})">चेक-इन के लिए क्लिक करें</button>
                    `;
                } else {
                    const isTimeUp = roomData.status === 'timeup';
                    card.className = `room-card ${isTimeUp ? 'status-timeup' : 'status-occupied'}`;
                    
                    card.innerHTML = `
                        <div>
                            <div class="room-header">कमरा ${num}</div>
                            <div class="status-badge">${isTimeUp ? 'समय समाप्त / चेक-आउट करें' : 'कमरा बुक है'}</div>
                            <div class="info-text"><strong>आगमन:</strong> <br>${formatDate(roomData.checkInTime)}</div>
                            <div class="timer-text" id="timer-${num}">हिसाब लगाया जा रहा है...</div>
                        </div>
                        <button class="btn-red" onclick="checkout(${num})">चेक-आउट करें</button>
                    `;
                }
                gridElement.appendChild(card);
            });
            updateTimers(); // Immediate update to prevent flicker
        }

        // Check-in Modal Logic
        function openCheckInModal(roomNum) {
            selectedRoomForCheckin = roomNum;
            document.getElementById('modal-room-title').innerText = `कमरा ${roomNum} में चेक-इन`;
            document.getElementById('modal-current-time').innerText = new Date().toLocaleString('en-IN');
            
            // Set exact correct defaults to avoid adding overlapping hours
            document.getElementById('input-days').value = 1; // Default to 1 Day (24 hours)
            document.getElementById('input-hours').value = 0; // Default to 0 extra hours
            
            modalOverlay.style.display = 'flex';
        }

        function closeModal() {
            modalOverlay.style.display = 'none';
            selectedRoomForCheckin = null;
        }

        function confirmCheckIn() {
            const days = parseInt(document.getElementById('input-days').value) || 0;
            const hours = parseInt(document.getElementById('input-hours').value) || 0;
            
            if (days === 0 && hours === 0) {
                alert('कृपया 0 से ज्यादा का समय डालें।');
                return;
            }

            // Calculation strictly converts 1 day = 24 hours. 
            const totalMs = (days * 24 * 60 * 60 * 1000) + (hours * 60 * 60 * 1000);
            const now = Date.now();
            
            let durationText = [];
            if (days > 0) durationText.push(`${days} दिन`);
            if (hours > 0) durationText.push(`${hours} घंटे`);

            rooms[selectedRoomForCheckin] = {
                status: 'occupied',
                checkInTime: now,
                endTime: now + totalMs,
                bookedDurationText: durationText.join(' और ')
            };

            saveState();
            closeModal();
            renderGrid();
        }

        // Check-out Logic
        function checkout(roomNum) {
            if(!confirm(`क्या आप वाकई कमरा ${roomNum} खाली (चेक-आउट) करना चाहते हैं?`)) return;

            const roomData = rooms[roomNum];
            
            // Log the history
            logs.unshift({ // Add to beginning of array
                room: roomNum,
                checkInTime: roomData.checkInTime,
                checkOutTime: Date.now(),
                duration: roomData.bookedDurationText
            });

            // Reset room
            rooms[roomNum] = {
                status: 'available',
                checkInTime: null,
                endTime: null,
                bookedDurationText: ''
            };

            saveState();
            renderGrid();
            renderLogs();
        }

        // Timer Loop logic
        function updateTimers() {
            const now = Date.now();
            let needsRender = false;

            ROOM_NUMBERS.forEach(num => {
                const roomData = rooms[num];
                if (roomData.status === 'available') return;

                const timerElement = document.getElementById(`timer-${num}`);
                if (!timerElement) return;

                const timeLeftMs = roomData.endTime - now;

                if (timeLeftMs <= 0) {
                    if (roomData.status !== 'timeup') {
                        // Time just expired, update state and trigger full re-render for colors
                        roomData.status = 'timeup';
                        saveState();
                        needsRender = true;
                    } else {
                        timerElement.innerText = "समय खत्म हो गया!";
                    }
                } else {
                    // Calculate H, M, S strictly mathematically
                    const h = Math.floor(timeLeftMs / (1000 * 60 * 60));
                    const m = Math.floor((timeLeftMs % (1000 * 60 * 60)) / (1000 * 60));
                    const s = Math.floor((timeLeftMs % (1000 * 60)) / 1000);
                    
                    timerElement.innerText = `बचा हुआ समय: ${h} घंटे ${m} मिनट ${s} सेकंड`;
                }
            });

            if (needsRender) renderGrid();
        }

        // Start background interval
        setInterval(updateTimers, 1000);

        // Logs Display Logic
        function toggleLogs() {
            if (logsSection.style.display === 'block') {
                logsSection.style.display = 'none';
            } else {
                renderLogs();
                logsSection.style.display = 'block';
                logsSection.scrollIntoView({ behavior: 'smooth' });
            }
        }

        function renderLogs() {
            logsBody.innerHTML = '';
            if (logs.length === 0) {
                logsBody.innerHTML = `<tr><td colspan="4" style="text-align: center;">अभी तक कोई पुराना रिकॉर्ड नहीं है।</td></tr>`;
                return;
            }
            logs.forEach(log => {
                const tr = document.createElement('tr');
                tr.innerHTML = `
                    <td>कमरा ${log.room}</td>
                    <td>${formatDate(log.checkInTime)}</td>
                    <td>${formatDate(log.checkOutTime)}</td>
                    <td>${log.duration}</td>
                `;
                logsBody.appendChild(tr);
            });
        }

        // Initial Boot
        renderGrid();
    </script>
</body>
</html>

```
