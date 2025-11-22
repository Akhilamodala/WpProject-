<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Calendar</title>

    <style>
        body {
            font-family: Arial, sans-serif;
            background: #eef2f3;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }

        /* PAGE 1 */
        #welcome-page {
            text-align: center;
        }
        #welcome-page img {
            width: 120px;
            margin-bottom: 20px;
        }

        /* PAGE 2 */
        #calendar-page {
            display: none;
            flex-direction: column;
            align-items: center;
        }

        h1 {
            margin-bottom: 20px;
        }

        .calendar {
            width: 500px;   /* UPDATED WIDTH */
            background: #fff;
            padding: 20px;
            border-radius: 12px;
            box-shadow: 0 0 15px rgba(0,0,0,0.1);
            position: relative;
        }

        .calendar-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 20px;
        }

        .btn-text {
            font-size: 20px;
            cursor: pointer;
            padding: 5px 10px;
            border-radius: 6px;
            transition: 0.3s;
            user-select: none;
        }

        .btn-text:hover {
            background: #007bff;
            color: white;
        }

        .days {
            display: grid;
            grid-template-columns: repeat(7, 1fr);
            text-align: center;
            font-weight: bold;
            margin-bottom: 10px;
        }

        .dates {
            display: grid;
            grid-template-columns: repeat(7, 1fr);
            text-align: center;
            gap: 5px;
        }

        .dates div {
            padding: 12px;
            border-radius: 5px;
            min-height: 60px;
            cursor: pointer;
            position: relative;
        }

        .dates div:hover {
            background: #007bff;
            color: white;
        }

        .today {
            background: #007bff;
            color: white;
            font-weight: bold;
        }

        .event-label {
            margin-top: 5px;
            font-size: 10px;
            background: #28a745;
            color: white;
            padding: 2px 4px;
            border-radius: 4px;
            display: block;
        }

        .popup {
            display: none;
            position: absolute;
            top: 60px;
            width: 100%;
            background: white;
            padding: 15px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0,0,0,0.2);
            max-height: 200px;
            overflow-y: auto;
        }

        .popup-grid {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 10px;
            text-align: center;
        }

        .popup-grid div {
            padding: 10px;
            border-radius: 6px;
            cursor: pointer;
            transition: 0.2s;
        }
        .popup-grid div:hover {
            background: #007bff;
            color: white;
        }

        /* FULL SCREEN EVENT EDITOR */
        #event-panel {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: white;
            display: none;
            padding: 30px;
            z-index: 1000;
        }

        #event-panel input {
            width: 100%;
            padding: 12px;
            margin-top: 15px;
            font-size: 16px;
            border-radius: 6px;
            border: 1px solid #888;
        }

        #event-panel button {
            margin-top: 20px;
            padding: 12px;
            width: 100%;
            background: #007bff;
            color: white;
            font-size: 18px;
            border: none;
            border-radius: 6px;
            cursor: pointer;
        }

        #delete-btn {
            background: red;
        }

        #close-btn {
            margin-top: 10px;
            background: gray;
        }
    </style>
</head>

<body>

    <!-- PAGE 1 -->
    <div id="welcome-page">
        <img src="https://cdn-icons-png.flaticon.com/512/747/747310.png">
        <h1>Calendar</h1>
    </div>

    <!-- PAGE 2 -->
    <div id="calendar-page">
        <h1>Calendar</h1>

        <div class="calendar">

            <div class="calendar-header">
                <div class="btn-text" id="prev-btn">◀</div>

                <div style="display:flex; gap:10px;">
                    <div class="btn-text" id="month-btn"></div>
                    <div class="btn-text" id="year-btn"></div>
                </div>

                <div class="btn-text" id="next-btn">▶</div>
            </div>

            <div class="days">
                <div>Sun</div><div>Mon</div><div>Tue</div><div>Wed</div>
                <div>Thu</div><div>Fri</div><div>Sat</div>
            </div>

            <div class="dates" id="dates"></div>

            <div class="popup" id="month-popup">
                <div class="popup-grid" id="month-grid"></div>
            </div>

            <div class="popup" id="year-popup">
                <div class="popup-grid" id="year-grid"></div>
            </div>

        </div>
    </div>

    <!-- EVENT PANEL -->
    <div id="event-panel">
        <h2 id="event-date-title"></h2>

        <input id="event-title" type="text" placeholder="Event Name">
        <input id="event-time" type="time">

        <button id="save-btn">Save Event</button>
        <button id="delete-btn" style="display:none;">Delete Event</button>
        <button id="close-btn">Cancel</button>
    </div>


<script>
    setTimeout(() => {
        document.getElementById("welcome-page").style.display = "none";
        document.getElementById("calendar-page").style.display = "flex";
    }, 2000);

    const datesContainer = document.getElementById("dates");
    const monthBtn = document.getElementById("month-btn");
    const yearBtn = document.getElementById("year-btn");

    const prevBtn = document.getElementById("prev-btn");
    const nextBtn = document.getElementById("next-btn");

    const monthPopup = document.getElementById("month-popup");
    const yearPopup = document.getElementById("year-popup");

    const monthGrid = document.getElementById("month-grid");
    const yearGrid = document.getElementById("year-grid");

    let currentDate = new Date();

    const monthNames = [
        "January","February","March","April","May","June",
        "July","August","September","October","November","December"
    ];

    let events = JSON.parse(localStorage.getItem("events") || "{}");

    monthNames.forEach((month, index) => {
        const div = document.createElement("div");
        div.textContent = month;
        div.onclick = () => {
            currentDate.setMonth(index);
            monthPopup.style.display = "none";
            renderCalendar();
        };
        monthGrid.appendChild(div);
    });

    for (let y = 1950; y <= 2050; y++) {
        const div = document.createElement("div");
        div.textContent = y;
        div.onclick = () => {
            currentDate.setFullYear(y);
            yearPopup.style.display = "none";
            renderCalendar();
        };
        yearGrid.appendChild(div);
    }

    function renderCalendar() {
        const year = currentDate.getFullYear();
        const month = currentDate.getMonth();

        monthBtn.textContent = monthNames[month];
        yearBtn.textContent = year;

        const firstDay = new Date(year, month, 1).getDay();
        const lastDate = new Date(year, month + 1, 0).getDate();

        datesContainer.innerHTML = "";

        for (let i = 0; i < firstDay; i++)
            datesContainer.innerHTML += "<div></div>";

        for (let day = 1; day <= lastDate; day++) {
            const dateKey = `${year}-${month + 1}-${day}`;
            const eventText = events[dateKey] ? events[dateKey].title : "";

            const div = document.createElement("div");

            const isToday =
                day === new Date().getDate() &&
                month === new Date().getMonth() &&
                year === new Date().getFullYear();

            if (isToday) div.classList.add("today");

            div.innerHTML = `${day} 
                ${eventText ? `<span class='event-label'>${eventText}</span>` : ""}`;

            div.onclick = () => openEventPanel(dateKey);

            datesContainer.appendChild(div);
        }
    }

    const eventPanel = document.getElementById("event-panel");
    const eventDateTitle = document.getElementById("event-date-title");
    const eventTitle = document.getElementById("event-title");
    const eventTime = document.getElementById("event-time");
    const saveBtn = document.getElementById("save-btn");
    const deleteBtn = document.getElementById("delete-btn");
    const closeBtn = document.getElementById("close-btn");

    let activeDate = "";

    function openEventPanel(dateKey) {
        activeDate = dateKey;

        eventDateTitle.textContent = "Event for: " + dateKey;

        if (events[dateKey]) {
            eventTitle.value = events[dateKey].title;
            eventTime.value = events[dateKey].time;
            deleteBtn.style.display = "block";
        } else {
            eventTitle.value = "";
            eventTime.value = "";
            deleteBtn.style.display = "none";
        }

        eventPanel.style.display = "block";
    }

    saveBtn.onclick = () => {
        if (eventTitle.value.trim() === "") {
            alert("Event title required");
            return;
        }

        events[activeDate] = {
            title: eventTitle.value,
            time: eventTime.value
        };

        localStorage.setItem("events", JSON.stringify(events));
        eventPanel.style.display = "none";
        renderCalendar();
    };

    deleteBtn.onclick = () => {
        delete events[activeDate];

        localStorage.setItem("events", JSON.stringify(events));
        eventPanel.style.display = "none";
        renderCalendar();
    };

    closeBtn.onclick = () => {
        eventPanel.style.display = "none";
    };

    prevBtn.onclick = () => {
        currentDate.setMonth(currentDate.getMonth() - 1);
        renderCalendar();
    };
    nextBtn.onclick = () => {
        currentDate.setMonth(currentDate.getMonth() + 1);
        renderCalendar();
    };

    monthBtn.onclick = () => {
        monthPopup.style.display = monthPopup.style.display === "none" ? "block" : "none";
        yearPopup.style.display = "none";
    };

    yearBtn.onclick = () => {
        yearPopup.style.display = yearPopup.style.display === "none" ? "block" : "none";
        monthPopup.style.display = "none";
    };

    document.addEventListener("click", (e) => {
        if (!monthBtn.contains(e.target) && !monthPopup.contains(e.target))
            monthPopup.style.display = "none";

        if (!yearBtn.contains(e.target) && !yearPopup.contains(e.target))
            yearPopup.style.display = "none";
    });

    renderCalendar();
</script>

</body>
</html>
