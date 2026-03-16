<!DOCTYPE html>
<html lang="bn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Future Link - Manual Laser Entry Map</title>
    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-database-compat.js"></script>
    <style>
        body { font-family: 'Segoe UI', Arial, sans-serif; background: #f0f2f5; padding: 10px; margin: 0; }
        .app-container { max-width: 900px; margin: auto; background: #fff; padding: 20px; border-radius: 12px; box-shadow: 0 5px 15px rgba(0,0,0,0.2); }
        h2 { text-align: center; color: #1a73e8; border-bottom: 2px solid #1a73e8; padding-bottom: 10px; margin-top: 0; }
        .brand-name { color: #ff5722; font-weight: bold; }
        .section { background: #f8f9fa; padding: 15px; border-radius: 8px; margin-bottom: 15px; border-left: 5px solid #1a73e8; }
        label { display: block; margin-bottom: 5px; font-weight: bold; font-size: 13px; color: #333; }
        select, input { width: 100%; padding: 10px; margin-bottom: 8px; border: 1px solid #ddd; border-radius: 6px; box-sizing: border-box; font-size: 14px; }
        
        /* লেজার বক্সের জন্য বিশেষ স্টাইল */
        .laser-input { background: #fffde7; border: 1.5px solid #fbc02d !important; font-weight: bold; color: #d32f2f; }
        
        .port-box { background: #fff; border: 1px solid #eee; padding: 12px; border-radius: 8px; border-left: 4px solid #28a745; margin-bottom: 12px; }
        .btn-save { width: 100%; padding: 15px; background: #28a745; color: white; border: none; border-radius: 8px; font-size: 18px; cursor: pointer; font-weight: bold; }
        .btn-view { background: #1a73e8; color: white; border: none; padding: 6px 10px; border-radius: 4px; cursor: pointer; }
        .btn-edit { background: #ffc107; color: black; border: none; padding: 6px 10px; border-radius: 4px; cursor: pointer; }
        .btn-delete { background: #dc3545; color: white; border: none; padding: 6px 10px; border-radius: 4px; cursor: pointer; }
        
        .modal { display: none; position: fixed; z-index: 1000; left: 0; top: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.8); }
        .modal-content { background: #fff; margin: 5% auto; padding: 25px; width: 90%; max-width: 550px; border-radius: 12px; position: relative; max-height: 85vh; overflow-y: auto; }
        .close-btn { position: absolute; right: 15px; top: 10px; font-size: 30px; cursor: pointer; }
        
        table { width: 100%; border-collapse: collapse; margin-top: 20px; font-size: 13px; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: center; }
        th { background: #1a73e8; color: white; }
        
        .view-row { border-bottom: 1px solid #eee; padding: 8px 0; }
        .color-tag { font-weight: bold; font-size: 11px; border: 1px solid #ccc; padding: 2px 6px; border-radius: 4px; background: #fff; display: inline-block; }
        .laser-tag { color: #d32f2f; font-weight: bold; }
    </style>
</head>
<body>

<div class="app-container">
    <h2><span class="brand-name">Future Link</span> - ডিজিটাল লেজার রেকর্ডার</h2>

    <input type="hidden" id="editKey" value="">

    <div class="section" style="border-left-color: #ff5722;">
        <label>🔗 সোর্স বক্স (কোন বক্স থেকে লাইন এসেছে?)</label>
        <input type="text" id="sourceBox" placeholder="যেমন: OLT বা TJ-01">
    </div>

    <div class="section">
        <label>📍 বর্তমান TJ বক্স নম্বর</label>
        <input type="text" id="tjLocation" placeholder="যেমন: TJ-02">
        
        <label>🎨 ইনপুট কোর (In Color)</label>
        <select id="mainCore" class="color-list"></select>
        
        <label>⚡ ইনপুট লেজার (dBm) - [হাতে লিখুন]</label>
        <input type="text" id="laserIn" class="laser-input" placeholder="যেমন: -18.50">
    </div>

    <div class="section">
        <label>⚙️ স্প্লিটার টাইপ</label>
        <select id="splitterType" onchange="generatePorts()">
            <option value="0">সরাসরি (No Splitter)</option>
            <option value="2">1:2 Splitter</option>
            <option value="4">1:4 Splitter</option>
            <option value="8">1:8 Splitter</option>
            <option value="16">1:16 Splitter</option>
        </select>
    </div>

    <div id="dynamicPortsSection" class="section" style="display:none;">
        <label>🚀 আউটপুট পোর্টগুলোর তথ্য</label>
        <div id="portList"></div>
    </div>

    <div id="directDestSection" class="section">
        <label>🚀 লাইন গন্তব্য</label>
        <select id="lineDest" onchange="toggleDirectInputs()">
            <option value="সোজা">সোজা গেছে (Main Line)</option>
            <option value="গ্রাহক">গ্রাহকের বাসা</option>
            <option value="নেক্সট">পরের টিজে বক্সে</option>
            <option value="স্প্লিটার">২য় স্প্লিটার (Cascade)</option>
        </select>
        <div id="directInputs" class="port-box" style="background:#eef2f7;">
            <div id="infoGroup" style="display:none;">
                <label>বিস্তারিত তথ্য (নাম/বক্স/সার্ভিস)</label>
                <input type="text" id="directInfo" placeholder="নাম বা সার্ভিস লাইন তথ্য">
                
                <label>⚡ আউটপুট লেজার (dBm) - [হাতে লিখুন]</label>
                <input type="text" id="directLaser" class="laser-input" placeholder="যেমন: -21.40">
            </div>
            <label>🎨 আউট কোর কালার</label>
            <select id="directOutColor" class="color-list"></select>
        </div>
    </div>

    <button class="btn-save" id="saveBtn" onclick="saveToFirebase()">ক্লাউডে সেভ করুন</button>

    <div class="section">
        <input type="text" id="searchInput" oninput="searchTable()" placeholder="সার্চ করুন (বক্স/নাম)...">
        <table id="resultTable">
            <thead>
                <tr><th>বক্স নং</th><th>টাইপ</th><th>অ্যাকশন</th></tr>
            </thead>
            <tbody></tbody>
        </table>
    </div>
</div>

<div id="viewModal" class="modal"><div class="modal-content"><span class="close-btn" onclick="closeModal()">&times;</span><div id="modalBody"></div></div></div>

<script>
    // Firebase Config
    const firebaseConfig = {
        apiKey: "AIzaSyByesWdI8Vpzd9sG4MIL51F07EdJYAksIk",
        authDomain: "futurelink-8d36c.firebaseapp.com",
        projectId: "futurelink-8d36c",
        storageBucket: "futurelink-8d36c.appspot.com",
        messagingSenderId: "309331499541",
        appId: "1:309331499541:web:ccfc5568e1abb304bdcc1c",
        databaseURL: "https://futurelink-8d36c-default-rtdb.firebaseio.com"
    };

    firebase.initializeApp(firebaseConfig);
    const db = firebase.database().ref("tj_records");
    const fiberColors = ["Blue", "Orange", "Green", "Brown", "Slate", "White", "Red", "Black", "Yellow", "Violet", "Pink", "Aqua"];

    function init() {
        document.querySelectorAll('.color-list').forEach(drop => {
            fiberColors.forEach(c => { drop.innerHTML += `<option value="${c}">${c}</option>`; });
        });
        db.on("value", snapshot => renderTable(snapshot.val()));
    }

    function toggleDirectInputs() {
        const dest = document.getElementById('lineDest').value;
        document.getElementById('infoGroup').style.display = (dest !== "সোজা") ? "block" : "none";
    }

    function generatePorts() {
        const type = parseInt(document.getElementById('splitterType').value);
        const pSection = document.getElementById('dynamicPortsSection');
        const dSection = document.getElementById('directDestSection');
        const pList = document.getElementById('portList');
        pList.innerHTML = "";
        
        if(type > 0) {
            pSection.style.display = "block"; dSection.style.display = "none";
            for(let i=1; i<=type; i++) {
                pList.innerHTML += `
                    <div class="port-box dynamic-port-item">
                        <label>পোর্ট P-${i} গন্তব্য</label>
                        <select class="p-dest">
                            <option value="গ্রাহক">গ্রাহক</option>
                            <option value="নেক্সট">নেক্সট বক্স</option>
                            <option value="স্প্লিটার">নতুন স্প্লিটার</option>
                            <option value="সোজা">সার্ভিস লাইন</option>
                        </select>
                        <label>নাম / তথ্য</label>
                        <input type="text" class="p-info" placeholder="গ্রাহকের নাম বা তথ্য">
                        
                        <label>⚡ আউটপুট লেজার (dBm) - [হাতে লিখুন]</label>
                        <input type="text" class="p-laser laser-input" placeholder="-22.5">
                        
                        <label>🎨 আউট কোর কালার</label>
                        <select class="p-color">${fiberColors.map(c => `<option value="${c}">${c}</option>`).join('')}</select>
                    </div>`;
            }
        } else { pSection.style.display = "none"; dSection.style.display = "block"; }
    }

    function saveToFirebase() {
        const key = document.getElementById('editKey').value;
        const data = {
            source: document.getElementById('sourceBox').value || "N/A",
            tj: document.getElementById('tjLocation').value || "Unnamed",
            laserIn: document.getElementById('laserIn').value || "0.00",
            splitter: document.getElementById('splitterType').value,
            mainColor: document.getElementById('mainCore').value,
            ports: []
        };

        if(data.splitter === "0") {
            data.direct = {
                dest: document.getElementById('lineDest').value,
                info: document.getElementById('directInfo').value || "",
                laserOut: document.getElementById('directLaser').value || "0.00",
                outColor: document.getElementById('directOutColor').value
            };
        } else {
            document.querySelectorAll('.dynamic-port-item').forEach(p => {
                data.ports.push({
                    dest: p.querySelector('.p-dest').value,
                    info: p.querySelector('.p-info').value,
                    laserOut: p.querySelector('.p-laser').value || "0.00",
                    color: p.querySelector('.p-color').value
                });
            });
        }

        if(key) {
            db.child(key).set(data).then(() => { alert("সফলভাবে আপডেট হয়েছে!"); resetForm(); });
        } else {
            db.push(data).then(() => { alert("সফলভাবে সেভ হয়েছে!"); resetForm(); });
        }
    }

    function renderTable(data) {
        const tbody = document.querySelector('#resultTable tbody');
        tbody.innerHTML = "";
        if(!data) return;
        Object.keys(data).forEach(key => {
            const item = data[key];
            const row = tbody.insertRow(0);
            row.innerHTML = `<td>${item.tj}</td><td>${item.splitter=='0'?'সরাসরি':'1:'+item.splitter}</td>
            <td>
                <button class="btn-view" onclick="viewDetails('${key}')">ভিউ</button> 
                <button class="btn-edit" onclick="loadEdit('${key}')">এডিট</button>
                <button class="btn-delete" onclick="deleteRecord('${key}')">মুছুন</button>
            </td>`;
        });
    }

    function loadEdit(key) {
        db.child(key).once("value", snapshot => {
            const item = snapshot.val();
            document.getElementById('editKey').value = key;
            document.getElementById('sourceBox').value = item.source;
            document.getElementById('tjLocation').value = item.tj;
            document.getElementById('laserIn').value = item.laserIn;
            document.getElementById('mainCore').value = item.mainColor;
            document.getElementById('splitterType').value = item.splitter;
            generatePorts();
            if(item.splitter === "0") {
                document.getElementById('lineDest').value = item.direct.dest;
                document.getElementById('directInfo').value = item.direct.info;
                document.getElementById('directLaser').value = item.direct.laserOut;
                document.getElementById('directOutColor').value = item.direct.outColor;
                toggleDirectInputs();
            } else {
                const ports = document.querySelectorAll('.dynamic-port-item');
                item.ports.forEach((p, i) => {
                    ports[i].querySelector('.p-dest').value = p.dest;
                    ports[i].querySelector('.p-info').value = p.info;
                    ports[i].querySelector('.p-laser').value = p.laserOut;
                    ports[i].querySelector('.p-color').value = p.color;
                });
            }
            document.getElementById('saveBtn').innerText = "আপডেট করুন";
            window.scrollTo(0,0);
        });
    }

    function viewDetails(key) {
        db.child(key).once("value", snapshot => {
            const item = snapshot.val();
            let html = `<h3>📦 বিস্তারিত রিপোর্ট</h3>
            <b>🔗 সোর্স:</b> ${item.source}<br><b>📍 বক্স:</b> ${item.tj}<br>
            <b>🎨 ইনপুট:</b> <span class="color-tag">${item.mainColor}</span><br>
            <b>⚡ ইন লেজার:</b> <span class="laser-tag">${item.laserIn} dBm</span><hr>`;
            
            if(item.splitter === "0") {
                html += `<b>🚀 গন্তব্য:</b> ${item.direct.dest}<br><b>📝 তথ্য:</b> ${item.direct.info}<br>
                <b>⚡ আউট লেজার:</b> <span class="laser-tag">${item.direct.laserOut} dBm</span><br>
                <b>🎨 আউট কালার:</b> <span class="color-tag">${item.direct.outColor}</span>`;
            } else {
                item.ports.forEach((p, i) => {
                    html += `<div class="view-row"><b>P${i+1}:</b> [${p.dest}] ${p.info}<br>
                    ⚡ আউট লেজার: <span class="laser-tag">${p.laserOut} dBm</span><br>
                    🎨 আউট কালার: <span class="color-tag">${p.color}</span></div>`;
                });
            }
            document.getElementById('modalBody').innerHTML = html;
            document.getElementById('viewModal').style.display = "block";
        });
    }

    function resetForm() {
        document.getElementById('editKey').value = "";
        document.getElementById('tjLocation').value = "";
        document.getElementById('laserIn').value = "";
        document.getElementById('splitterType').value = "0";
        document.getElementById('saveBtn').innerText = "ক্লাউডে সেভ করুন";
        generatePorts();
        toggleDirectInputs();
    }

    function deleteRecord(key) { if(confirm("মুছে ফেলবেন?")) db.child(key).remove(); }
    function closeModal() { document.getElementById('viewModal').style.display = "none"; }
    function searchTable() {
        let val = document.getElementById("searchInput").value.toLowerCase();
        let rows = document.querySelectorAll("#resultTable tbody tr");
        rows.forEach(r => { r.style.display = r.innerText.toLowerCase().includes(val) ? "" : "none"; });
    }
    window.onload = init;
</script>
</body>
    </html>
