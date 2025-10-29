# Naija-Catholic-Parish-Assistant-ncpa-

## Problem & Business Value
Across Nigeria, Catholics face a silent but widespread challenge, finding the sacraments when they are away from their home parish. Whether someone is travelling for work, schooling in a new city, visiting relatives, on a business trip, or simply passing through an unfamiliar environment, there is no reliable or unified way to know where and when Mass or Confession is available nearby. Most parishes operate independently with no shared digital structure, so the ordinary Catholic is left to depend on guesswork, word-of-mouth, outdated Facebook posts, parish signboards, or WhatsApp groups that they are not even a part of. As a result, people often miss the sacraments not because they are unwilling, but because the information is hidden.
The difficulty is greatest for travelers, students living outside their home diocese, NYSC corps members, and Catholics temporarily in other cities. Finding Mass becomes a matter of luck: asking strangers, calling friends to “see if they know a parish around,” or physically walking around until a church is found. Confession is even harder. There is rarely a public schedule anywhere online. Even priests face the opposite side of the problem: there is no central platform where their official schedules can be seen or updated, so and the diocese has no real-time visibility of sacramental availability across its parishes.
This is not a faith problem; it is an information access problem.
The purpose of this project is to solve that gap by making sacramental access instant and location-based. Instead of searching blindly, a Catholic should be able to open one app and immediately see the top nearest Masses, Confession times, or priest availability around them, arranged by distance and time. In doing so, the Church becomes visible and reachable wherever the faithful are, not only where they are registered.
By simplifying the core features of the system to just three essentials; Mass schedules, Confession times, and priest availability, the app becomes fast, lightweight, and easy to manage for administrators, while remaining spiritually life-changing for users. This structure creates the foundation upon which AI and other pastoral features can later be built, but the heart of the platform remains the same: restoring convenient access to the sacraments through structured, real-time information.

## Capabilities of the App
This app is a real-time sacrament locator for Catholics on the move. It removes guesswork by showing nearby parishes with Mass, Confession, or priest counselling available now or next, based on GPS. Alongside this core utility, it also presents the Catholic Daily Readings so users can pray with the Church’s liturgy each day without leaving the app.
Core Capabilities

i.	Nearest Mass Finder: Instantly lists parishes having Mass now or starting soon, ranked by distance and start time.

ii.	Nearest Confession Finder: Shows where Confession is available today and the next upcoming slots.

iii.	Priest Availability (Counselling Hours): Surfaces when/where a priest is available for spiritual direction.

iv.	Daily Readings (Free API, No DB storage): Displays Today’s readings (First Reading, Psalm, Gospel) fetched live from a free Catholic API—no extra tables or maintenance.

v.	Distance- & Time-Aware Results: Combines GPS with schedules to prioritize what’s both close and timely.

vi.	Immediate Access UI + Optional AI: Home screen shows lists immediately (top 10 nearby). A button lets users “Ask AI” for guided help if they prefer conversation.





## Technology Stack & Rationale

| Technology | Role | Reason |
|-----------|------|--------|
| **PostgreSQL** | Stores parish schedules | Structured, scalable, and ideal for managing time-based queries |
| **n8n** | Orchestrates business logic & exposes API endpoints | Faster and more flexible than building a custom backend from scratch |
| **Free Catholic Daily Readings API** | Supplies daily liturgy | Eliminates need for manual data entry or storage of mass readings |
| **Flutter** | Mobile application layer | Single codebase for Android & iOS reduces development and maintenance time |
| **GPS + Google Maps** | Locates nearest parish & provides directions | Ensures accurate geolocation, routing, and navigation for parishioners |

## Project Implementation Plan

### STEP 1 – Database Foundation
In this step we will create the core database structure that will hold parish information and sacramental schedules. Only four essential tables will be created: one for parishes, one for all Mass schedules (weekday and Sunday combined), one for Confession times, and one for priest availability. This step prepares the backbone of the entire system, because every future function will depend on clean and well-structured time-based sacramental data.


i.	In your already created postgre database, let us create a registration table. Paste the below on your db
```
CREATE TABLE parish_registration (
    parish_name VARCHAR(255) NOT NULL,
    parish_address TEXT NOT NULL,
    state VARCHAR(100) NOT NULL,
    local_government VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    phone_number VARCHAR(20) NOT NULL,
    password VARCHAR(255) NOT NULL
);
``` 
<img width="867" height="300" alt="image" src="https://github.com/user-attachments/assets/7a8081e1-dc31-458e-9fbf-df1ce25eb6fa" />


ii.	Create the Masses table (weekday + Sunday together). Paste the below;
```
CREATE TABLE masses (
    parish_email VARCHAR(255) NOT NULL,
    day_of_week VARCHAR(20) NOT NULL,  -- Monday, Tuesday ... Sunday
    start_time VARCHAR(20) NOT NULL,   -- 12-hour format input (e.g. 6:30 AM)
    end_time VARCHAR(20) NOT NULL,
    created_at TIMESTAMPTZ DEFAULT now(),
    updated_at TIMESTAMPTZ DEFAULT now(),
    FOREIGN KEY (parish_email) REFERENCES parish_registration(email) ON DELETE CASCADE
);

CREATE INDEX idx_masses_parish ON masses (parish_email, day_of_week);
``` 
<img width="975" height="299" alt="image" src="https://github.com/user-attachments/assets/15470665-21e2-47d2-996f-dc726c743ffb" />


iii.	Create Confessions and Priest Availability tables. Paste the below;
```
-- Confession Schedules Table
CREATE TABLE confessions (
    parish_email VARCHAR(255) NOT NULL,
    day_of_week VARCHAR(20) NOT NULL,
    start_time VARCHAR(20) NOT NULL,
    end_time VARCHAR(20) NOT NULL,
    created_at TIMESTAMPTZ DEFAULT now(),
    updated_at TIMESTAMPTZ DEFAULT now(),
    FOREIGN KEY (parish_email) REFERENCES parish_registration(email) ON DELETE CASCADE
);

CREATE INDEX idx_confessions_parish ON confessions (parish_email, day_of_week);


-- Priest Availability Table
CREATE TABLE priest_availability (
    parish_email VARCHAR(255) NOT NULL,
    day_of_week VARCHAR(20) NOT NULL,
    start_time VARCHAR(20) NOT NULL,
    end_time VARCHAR(20) NOT NULL,
    created_at TIMESTAMPTZ DEFAULT now(),
    updated_at TIMESTAMPTZ DEFAULT now(),
    FOREIGN KEY (parish_email) REFERENCES parish_registration(email) ON DELETE CASCADE
);

CREATE INDEX idx_availability_parish ON priest_availability (parish_email, day_of_week);
```


iv.	To verify that everything was created correctly, run this command
```
\dt
``` 
<img width="936" height="314" alt="image" src="https://github.com/user-attachments/assets/18c75755-41d5-4f54-b882-b5bd28e5af8c" />

### STEP 2 – Admin Data Entry Layer
Once the database structure is ready, the next step is to provide a way for parishes or priests to enter their schedule information. This is where the admin portal or input layer comes in. Parish administrators will be able to log in and populate their Mass timings, Confession sessions, and priest counselling hours. After this step, the database stops being empty and begins holding real sacramental data.

i.	Create Admin Folder & Base Streamlit App. Run these commands one by one:
```
cd /root/projects/ncpc
mkdir admin
cd admin
touch app.py
```

ii.	Install Streamlit. It would be used for our Admin Panel. Run;
```
pip install streamlit psycopg2-binary
```

iii.	Open your app.py and paste the below code;

[admin_web_code](https://github.com/Ogbunugafor-Philip/Naija-Catholic-Parish-Assistant-ncpa-/blob/main/Admin%20Web%20Code)


iv.	View the admin webpage with the below address
```
http://<localhost>:8501/
```
<img width="975" height="378" alt="image" src="https://github.com/user-attachments/assets/8fb04eb8-5d9a-4b26-98a7-993493528cc1" />


Now that our admin page is created, let us keep it running constantly with the address http://<localhost>:8501/ for now

v.	Create the systemd service file. Run;
```
sudo vi /etc/systemd/system/ncpa.service
```

vi.	Paste the below configuration
```
[Unit]
Description=NCPA - Naija Catholic Parish Assistant
After=network.target postgresql.service

[Service]
Type=simple
User=root
WorkingDirectory=/root/projects/ncpc/admin
Environment="PATH=/usr/local/bin:/usr/bin:/bin"
ExecStart=/usr/local/bin/streamlit run app.py --server.port 8501 --server.address 0.0.0.0
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```
<img width="975" height="420" alt="image" src="https://github.com/user-attachments/assets/334fddfb-96af-44cf-9aeb-32764fcb92ac" />

vii.	Reload systemd and enable the service. Run these commands one by one
```
sudo systemctl daemon-reload
sudo systemctl enable ncpa.service
sudo systemctl start ncpa.service
sudo systemctl status ncpa.service
```
<img width="975" height="418" alt="image" src="https://github.com/user-attachments/assets/36729048-337c-4a7a-aabb-dfe29757c723" />




