<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Orange Dynasty Streak Tracker - Ready to Run</title>
<style>
body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: #fff7e6;
    color: #333;
    margin: 0;
    padding: 0;
    display: flex;
    justify-content: center;
}
.container {
    max-width: 800px;
    width: 100%;
    padding: 20px;
    text-align: center;
}
h1 { color: #ff6600; margin-bottom: 20px; }
.upload-section {
    background: #fff3e6;
    padding: 20px;
    border-radius: 15px;
    box-shadow: 0 4px 8px rgba(0,0,0,0.1);
    margin-bottom: 30px;
}
.upload-section input[type="text"],
.upload-section input[type="file"],
.upload-section input[type="number"] {
    margin: 10px 0;
    padding: 10px;
    width: 70%;
    border-radius: 10px;
    border: 1px solid #ccc;
}
button {
    padding: 10px 20px;
    background: #ff6600;
    color: white;
    border: none;
    border-radius: 10px;
    cursor: pointer;
    margin-top: 10px;
}
button:hover { background: #e65c00; }

.badge-section, .leaderboard-section { text-align: center; margin-top: 20px; }

.badge {
    display: inline-block;
    margin: 15px;
    padding: 10px;
    border-radius: 15px;
    background: #fff3e6;
    box-shadow: 2px 2px 5px rgba(0,0,0,0.1);
    position: relative;
}
.badge img { max-width: 200px; border-radius: 10px; border: 3px solid #ff6600; }
.badge p { margin: 5px 0 0 0; font-weight: bold; color: #ff6600; }
.badge button { margin: 5px 5px 0 5px; padding: 5px 10px; font-size: 12px; border-radius: 5px; }

.sparkle {
    position: absolute;
    width: 10px;
    height: 10px;
    background: #ff0;
    border-radius: 50%;
    opacity: 0.8;
    animation: sparkle 1s linear infinite;
}
@keyframes sparkle {
    0% { transform: scale(1) translate(0,0); opacity: 0.8; }
    50% { transform: scale(1.5) translate(5px,-5px); opacity: 0.3; }
    100% { transform: scale(1) translate(0,0); opacity: 0.8; }
}

.leaderboard-section table {
    margin: 0 auto;
    border-collapse: collapse;
    width: 90%;
}
.leaderboard-section th, .leaderboard-section td {
    border: 1px solid #ff6600;
    padding: 8px;
    text-align: center;
}
.leaderboard-section th { background: #ff6600; color: #fff; }
</style>
</head>
<body>
<div class="container">
<h1>🍊 Orange Dynasty Streak Tracker</h1>

<div class="upload-section">
    <input type="text" id="username" placeholder="Enter your username"><br>
    <input type="file" id="imageUpload" accept="image/*"><br>
    <input type="number" id="streakDays" placeholder="Enter streak days" min="1"><br>
    <button id="generateBadge">Generate Badge</button>
</div>

<div class="badge-section">
    <h2>Your Badges</h2>
    <div id="badgesContainer"></div>
</div>

<div class="leaderboard-section">
    <h2>Leaderboard 🏆</h2>
    <table id="leaderboardTable">
        <thead>
            <tr><th>Rank</th><th>Username</th><th>Streak Days</th></tr>
        </thead>
        <tbody></tbody>
    </table>
</div>

</div>

<script>
const usernameInput = document.getElementById('username');
const imageUpload = document.getElementById('imageUpload');
const streakDaysInput = document.getElementById('streakDays');
const generateBadgeBtn = document.getElementById('generateBadge');
const badgesContainer = document.getElementById('badgesContainer');
const leaderboardTable = document.getElementById('leaderboardTable').querySelector('tbody');

let badges = JSON.parse(localStorage.getItem('badges')) || [];
renderBadges();
renderLeaderboard();

generateBadgeBtn.addEventListener('click', () => {
    const username = usernameInput.value.trim();
    const file = imageUpload.files[0];
    const days = parseInt(streakDaysInput.value);

    if(!username || !file || !days || days <= 0){
        alert("Please enter a username, upload an image, and enter a valid streak!");
        return;
    }

    const reader = new FileReader();
    reader.onload = function(e){
        createBadgeImage(e.target.result, days, (badgeURL)=>{
            const badge = { username: username, img: badgeURL, days: days };
            badges.push(badge);
            localStorage.setItem('badges', JSON.stringify(badges));
            renderBadges();
            renderLeaderboard();
        });
    };
    reader.readAsDataURL(file);
});

function renderBadges(){
    badgesContainer.innerHTML='';
    badges.forEach((badge,index)=>{
        const badgeDiv=document.createElement('div');
        badgeDiv.classList.add('badge');

        let borderColor = '#ff6600';
        if(badge.days>=100) borderColor='#ff00ff';
        else if(badge.days>=50) borderColor='#00ffff';
        else if(badge.days>=30) borderColor='#e5e4e2';

        badgeDiv.innerHTML=`
            <img src="${badge.img}" alt="Streak Badge" style="border-color:${borderColor}">
            <p>${badge.username} - ${badge.days} Day${badge.days>1?'s':''} Streak</p>
            <button onclick="deleteBadge(${index})">Delete</button>
            <button onclick="downloadBadge('${badge.img}','${badge.username}_${badge.days}daystreak.png')">Download</button>
        `;

        if(badge.days>=100){
            for(let i=0;i<5;i++){
                const sparkle = document.createElement('div');
                sparkle.classList.add('sparkle');
                sparkle.style.top = Math.random()*150+'px';
                sparkle.style.left = Math.random()*200+'px';
                badgeDiv.appendChild(sparkle);
            }
        }

        badgesContainer.appendChild(badgeDiv);
    });
}

function deleteBadge(index){
    badges.splice(index,1);
    localStorage.setItem('badges',JSON.stringify(badges));
    renderBadges();
    renderLeaderboard();
}

function getBadgeColor(days){
    if(days>=100) return '#ff00ff';
    if(days>=50) return '#00ffff';
    if(days>=30) return '#e5e4e2';
    if(days>=15) return '#ffd700';
    if(days>=5) return '#c0c0c0';
    return '#cd7f32';
}

function createBadgeImage(imageSrc, days, callback){
    const canvas=document.createElement('canvas');
    const ctx=canvas.getContext('2d');
    const img=new Image();
    const icon=new Image();
    img.onload=()=>{
        canvas.width=img.width;
        canvas.height=img.height+70;
        ctx.shadowColor=getBadgeColor(days);
        ctx.shadowBlur=20;
        ctx.drawImage(img,0,0);
        ctx.shadowBlur=0;
        ctx.fillStyle=getBadgeColor(days);
        ctx.fillRect(0,0,canvas.width,50);
        ctx.fillStyle='#fff';
        ctx.font='bold 26px Arial';
        ctx.textAlign='center';
        ctx.fillText(`${days} Day${days>1?'s':''} Streak`, canvas.width/2,32);
        icon.onload=()=>{
            const iconSize=30;
            ctx.drawImage(icon,canvas.width-iconSize-10,10,iconSize,iconSize);
            callback(canvas.toDataURL('image/png'));
        };

        if(days>=100) icon.src='https://upload.wikimedia.org/wikipedia/commons/thumb/f/f4/Star_icon-72a7cf.svg/64px-Star_icon-72a7cf.svg.png';
        else if(days>=50) icon.src='https://upload.wikimedia.org/wikipedia/commons/thumb/5/55/Diamond_icon.png/64px-Diamond_icon.png';
        else if(days>=30) icon.src='https://upload.wikimedia.org/wikipedia/commons/thumb/1/16/Platinum_icon.png/64px-Platinum_icon.png';
        else icon.src='https://upload.wikimedia.org/wikipedia/commons/thumb/5/5a/Trophy_icon.png/64px-Trophy_icon.png';
    };
    img.src=imageSrc;
}

function downloadBadge(dataURL,filename){
    const link=document.createElement('a');
    link.href=dataURL;
    link.download=filename;
    link.click();
}

function renderLeaderboard(){
    leaderboardTable.innerHTML='';
    const sorted=[...badges].sort((a,b)=>b.days-a.days);
    sorted.forEach((badge,index)=>{
        const tr=document.createElement('tr');
        let color=''; 
        if(badge.days>=100) color='#ff00ff';
        else if(badge.days>=50) color='#00ffff';
        else if(badge.days>=30) color='#e5e4e2';
        else if(badge.days>=15) color='#ffd700';
        else if(badge.days>=5) color='#c0c0c0';
        else color='#cd7f32';
        tr.innerHTML=`<td>${index+1}</td><td style="color:${color}">${badge.username}</td><td>${badge.days}</td>`;
        leaderboardTable.appendChild(tr);
    });
}
</script>
</body>
</html>
