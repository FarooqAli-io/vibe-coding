# vibe-coding

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>🚀 Asteroid Blaster Ultimate</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            background: #000;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            overflow: hidden;
            font-family: 'Segoe UI', Arial, sans-serif;
        }
        canvas {
            border: 2px solid #1a3a5c;
            border-radius: 8px;
            box-shadow: 0 0 30px rgba(0, 100, 255, 0.3), inset 0 0 30px rgba(0, 0, 0, 0.5);
        }
    </style>
</head>
<body>
    <canvas id="gameCanvas" width="950" height="700"></canvas>
    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const CW = canvas.width, CH = canvas.height;

        // ============ AUDIO ============
        const AudioCtx = window.AudioContext || window.webkitAudioContext;
        let audioCtx;
        let masterVolume = 0.7, sfxVolume = 0.8;

        function initAudio() { if (!audioCtx) audioCtx = new AudioCtx(); }

        function playSound(type) {
            if (!audioCtx) return;
            try {
                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();
                osc.connect(gain); gain.connect(audioCtx.destination);
                let vol = masterVolume * sfxVolume;
                let t = audioCtx.currentTime;
                if (type === 'shoot') {
                    osc.type = 'square'; osc.frequency.setValueAtTime(800, t);
                    osc.frequency.exponentialRampToValueAtTime(200, t + 0.1);
                    gain.gain.setValueAtTime(0.1 * vol, t);
                    gain.gain.exponentialRampToValueAtTime(0.001, t + 0.1);
                    osc.start(); osc.stop(t + 0.1);
                } else if (type === 'explosion') {
                    osc.type = 'sawtooth'; osc.frequency.setValueAtTime(200, t);
                    osc.frequency.exponentialRampToValueAtTime(50, t + 0.3);
                    gain.gain.setValueAtTime(0.15 * vol, t);
                    gain.gain.exponentialRampToValueAtTime(0.001, t + 0.3);
                    osc.start(); osc.stop(t + 0.3);
                } else if (type === 'powerup') {
                    osc.type = 'sine'; osc.frequency.setValueAtTime(400, t);
                    osc.frequency.exponentialRampToValueAtTime(1200, t + 0.2);
                    gain.gain.setValueAtTime(0.1 * vol, t);
                    gain.gain.exponentialRampToValueAtTime(0.001, t + 0.25);
                    osc.start(); osc.stop(t + 0.25);
                } else if (type === 'hit') {
                    osc.type = 'triangle'; osc.frequency.setValueAtTime(600, t);
                    osc.frequency.exponentialRampToValueAtTime(100, t + 0.08);
                    gain.gain.setValueAtTime(0.08 * vol, t);
                    gain.gain.exponentialRampToValueAtTime(0.001, t + 0.08);
                    osc.start(); osc.stop(t + 0.08);
                } else if (type === 'gameover') {
                    osc.type = 'sawtooth'; osc.frequency.setValueAtTime(400, t);
                    osc.frequency.exponentialRampToValueAtTime(30, t + 1);
                    gain.gain.setValueAtTime(0.15 * vol, t);
                    gain.gain.exponentialRampToValueAtTime(0.001, t + 1);
                    osc.start(); osc.stop(t + 1);
                } else if (type === 'purchase') {
                    osc.type = 'sine'; osc.frequency.setValueAtTime(523, t);
                    osc.frequency.setValueAtTime(659, t + 0.1);
                    osc.frequency.setValueAtTime(784, t + 0.2);
                    gain.gain.setValueAtTime(0.12 * vol, t);
                    gain.gain.exponentialRampToValueAtTime(0.001, t + 0.35);
                    osc.start(); osc.stop(t + 0.35);
                } else if (type === 'error') {
                    osc.type = 'square'; osc.frequency.setValueAtTime(150, t);
                    osc.frequency.setValueAtTime(100, t + 0.1);
                    gain.gain.setValueAtTime(0.1 * vol, t);
                    gain.gain.exponentialRampToValueAtTime(0.001, t + 0.2);
                    osc.start(); osc.stop(t + 0.2);
                } else if (type === 'navigate') {
                    osc.type = 'sine'; osc.frequency.setValueAtTime(600, t);
                    gain.gain.setValueAtTime(0.05 * vol, t);
                    gain.gain.exponentialRampToValueAtTime(0.001, t + 0.06);
                    osc.start(); osc.stop(t + 0.06);
                } else if (type === 'select') {
                    osc.type = 'sine'; osc.frequency.setValueAtTime(800, t);
                    osc.frequency.setValueAtTime(1000, t + 0.05);
                    gain.gain.setValueAtTime(0.08 * vol, t);
                    gain.gain.exponentialRampToValueAtTime(0.001, t + 0.1);
                    osc.start(); osc.stop(t + 0.1);
                } else if (type === 'rebirth') {
                    osc.type = 'sine'; osc.frequency.setValueAtTime(300, t);
                    osc.frequency.exponentialRampToValueAtTime(1500, t + 0.5);
                    gain.gain.setValueAtTime(0.15 * vol, t);
                    gain.gain.exponentialRampToValueAtTime(0.001, t + 0.8);
                    osc.start(); osc.stop(t + 0.8);
                    // Second tone
                    const o2 = audioCtx.createOscillator();
                    const g2 = audioCtx.createGain();
                    o2.connect(g2); g2.connect(audioCtx.destination);
                    o2.type = 'sine'; o2.frequency.setValueAtTime(600, t + 0.3);
                    o2.frequency.exponentialRampToValueAtTime(2000, t + 0.8);
                    g2.gain.setValueAtTime(0.1 * vol, t + 0.3);
                    g2.gain.exponentialRampToValueAtTime(0.001, t + 1);
                    o2.start(t + 0.3); o2.stop(t + 1);
                } else if (type === 'enemyShoot') {
                    osc.type = 'sawtooth'; osc.frequency.setValueAtTime(400, t);
                    osc.frequency.exponentialRampToValueAtTime(150, t + 0.15);
                    gain.gain.setValueAtTime(0.06 * vol, t);
                    gain.gain.exponentialRampToValueAtTime(0.001, t + 0.15);
                    osc.start(); osc.stop(t + 0.15);
                } else if (type === 'boss') {
                    osc.type = 'sawtooth'; osc.frequency.setValueAtTime(80, t);
                    osc.frequency.setValueAtTime(120, t + 0.2);
                    osc.frequency.setValueAtTime(60, t + 0.4);
                    gain.gain.setValueAtTime(0.12 * vol, t);
                    gain.gain.exponentialRampToValueAtTime(0.001, t + 0.6);
                    osc.start(); osc.stop(t + 0.6);
                }
            } catch (e) {}
        }

        // ============ SAVE SYSTEM ============
        function defaultSave() {
            return {
                credits: 0, highScore: 0, totalGamesPlayed: 0,
                totalAsteroidsDestroyed: 0, totalEnemiesDestroyed: 0,
                totalDeaths: 0, highestWave: 0,
                rebirths: 0, rebirthTokens: 0, totalRebirthTokensEarned: 0,
                upgrades: {
                    hull: 0, fireRate: 0, speed: 0, startPower: 0,
                    shieldDuration: 0, critChance: 0, magnetRange: 0,
                    extraLives: 0, laserDamage: 0, laserPierce: 0,
                    multiShot: 0, spreadShot: 0, rearGun: 0,
                    homingMissiles: 0, scoreMultiplier: 0, luckyDrops: 0
                },
                rebirthUpgrades: {
                    permanentDamage: 0, permanentHP: 0, permanentSpeed: 0,
                    creditMultiplier: 0, startingCredits: 0, weaponMastery: 0,
                    enemyRadar: 0, cosmicArmor: 0, voidEnergy: 0,
                    temporalShift: 0
                },
                selectedShipSkin: 0,
                unlockedSkins: [true, false, false, false, false, false, false],
                settings: {
                    masterVolume: 0.7, sfxVolume: 0.8, musicEnabled: true,
                    screenShake: true, showFPS: false, particleDensity: 1
                }
            };
        }

        function loadSave() {
            try {
                let d = JSON.parse(localStorage.getItem('asteroidBlasterUltimate'));
                if (d) {
                    let def = defaultSave();
                    // Merge missing keys
                    for (let k in def) {
                        if (d[k] === undefined) d[k] = def[k];
                        if (typeof def[k] === 'object' && !Array.isArray(def[k])) {
                            for (let k2 in def[k]) {
                                if (d[k][k2] === undefined) d[k][k2] = def[k][k2];
                            }
                        }
                    }
                    return d;
                }
            } catch (e) {}
            return defaultSave();
        }

        function saveSave() { localStorage.setItem('asteroidBlasterUltimate', JSON.stringify(saveData)); }

        let saveData = loadSave();
        masterVolume = saveData.settings.masterVolume;
        sfxVolume = saveData.settings.sfxVolume;

        // ============ UPGRADE DEFINITIONS ============
        const UPGRADES = {
            hull: { name: 'Hull Plating', icon: '🛡️', desc: 'Increases maximum hull integrity', maxLevel: 10,
                costs: [200,400,800,1500,2500,4000,6000,9000,13000,18000],
                effect: l => `+${l*15} Max HP`, color: '#33ff66', category: 'defense' },
            fireRate: { name: 'Rapid Fire', icon: '🔥', desc: 'Decreases weapon cooldown', maxLevel: 10,
                costs: [300,600,1200,2000,3500,5500,8000,12000,16000,22000],
                effect: l => `${Math.round(l*10)}% faster`, color: '#ff6633', category: 'offense' },
            speed: { name: 'Thrusters', icon: '⚡', desc: 'Movement speed boost', maxLevel: 8,
                costs: [250,500,1000,2000,3500,5500,8000,12000],
                effect: l => `+${(l*0.5).toFixed(1)} speed`, color: '#ffff00', category: 'utility' },
            startPower: { name: 'Weapon Array', icon: '🔫', desc: 'Start at higher power level', maxLevel: 4,
                costs: [2000,5000,12000,25000],
                effect: l => `Start Power Lv.${l+1}`, color: '#ff1493', category: 'offense' },
            shieldDuration: { name: 'Shield Gen', icon: '💠', desc: 'Shield pickups last longer', maxLevel: 8,
                costs: [400,800,1500,3000,5000,8000,12000,18000],
                effect: l => `+${l*20}% duration`, color: '#00ffff', category: 'defense' },
            critChance: { name: 'Targeting CPU', icon: '🎯', desc: 'Chance for double damage', maxLevel: 10,
                costs: [500,1000,2000,4000,7000,10000,14000,19000,25000,32000],
                effect: l => `${l*5}% crit chance`, color: '#ff3333', category: 'offense' },
            magnetRange: { name: 'Tractor Beam', icon: '🧲', desc: 'Auto-collect power-ups', maxLevel: 6,
                costs: [350,700,1400,2800,5000,9000],
                effect: l => `${50+l*30}px range`, color: '#cc66ff', category: 'utility' },
            extraLives: { name: 'Escape Pods', icon: '💚', desc: 'Extra starting lives', maxLevel: 5,
                costs: [1500,5000,15000,35000,80000],
                effect: l => `${3+l} lives`, color: '#66ff66', category: 'defense' },
            laserDamage: { name: 'Laser Power', icon: '💥', desc: 'Each shot deals more damage', maxLevel: 10,
                costs: [400,800,1600,3200,6000,10000,15000,22000,30000,40000],
                effect: l => `+${l} base damage`, color: '#ff8800', category: 'offense' },
            laserPierce: { name: 'Piercing Rounds', icon: '🔱', desc: 'Lasers pass through enemies', maxLevel: 5,
                costs: [2000,5000,12000,25000,50000],
                effect: l => `Pierce ${l} targets`, color: '#00ff88', category: 'offense' },
            multiShot: { name: 'Multi-Shot', icon: '🌟', desc: 'Fire additional projectiles', maxLevel: 8,
                costs: [1000,2500,5000,10000,20000,35000,55000,80000],
                effect: l => `${getFireCount(l+1)} shots`, color: '#ffcc00', category: 'offense' },
            spreadShot: { name: 'Spread Angle', icon: '🔆', desc: 'Wider projectile spread', maxLevel: 6,
                costs: [800,1600,3200,6500,13000,25000],
                effect: l => `${10+l*8}° spread`, color: '#ff66ff', category: 'offense' },
            rearGun: { name: 'Rear Turret', icon: '🔙', desc: 'Fire backwards too', maxLevel: 3,
                costs: [3000,10000,30000],
                effect: l => `${l} rear gun${l>1?'s':''}`, color: '#66ccff', category: 'offense' },
            homingMissiles: { name: 'Homing Missiles', icon: '🚀', desc: 'Auto-tracking missiles', maxLevel: 5,
                costs: [5000,12000,25000,50000,100000],
                effect: l => `${l} missile${l>1?'s':''}/3s`, color: '#ff4400', category: 'offense' },
            scoreMultiplier: { name: 'Score Booster', icon: '✨', desc: 'Earn more points', maxLevel: 10,
                costs: [500,1000,2000,4000,8000,15000,25000,40000,60000,90000],
                effect: l => `+${l*10}% score`, color: '#ffff66', category: 'utility' },
            luckyDrops: { name: 'Lucky Drops', icon: '🍀', desc: 'Better power-up drop rate', maxLevel: 8,
                costs: [400,800,1600,3200,6000,10000,16000,24000],
                effect: l => `+${l*3}% drop rate`, color: '#33ff33', category: 'utility' }
        };

        function getFireCount(powerLevel) {
            if (powerLevel <= 1) return 1;
            if (powerLevel === 2) return 2;
            if (powerLevel === 3) return 3;
            if (powerLevel === 4) return 5;
            if (powerLevel === 5) return 5;
            if (powerLevel === 6) return 7;
            if (powerLevel === 7) return 7;
            if (powerLevel === 8) return 9;
            return Math.min(12, 3 + powerLevel);
        }

        // ============ REBIRTH UPGRADES ============
        const REBIRTH_UPGRADES = {
            permanentDamage: { name: 'Cosmic Firepower', icon: '⚔️', desc: 'Permanent +1 damage per level', maxLevel: 10,
                costs: [1,2,3,5,8,12,18,25,35,50], effect: l => `+${l} base damage`, color: '#ff4444' },
            permanentHP: { name: 'Stellar Constitution', icon: '❤️', desc: 'Permanent +25 max HP', maxLevel: 10,
                costs: [1,2,3,5,8,12,18,25,35,50], effect: l => `+${l*25} max HP`, color: '#ff6699' },
            permanentSpeed: { name: 'Warp Drive', icon: '🌀', desc: 'Permanent +0.3 speed', maxLevel: 8,
                costs: [1,2,4,6,10,15,22,30], effect: l => `+${(l*0.3).toFixed(1)} speed`, color: '#66ffff' },
            creditMultiplier: { name: 'Golden Nebula', icon: '💫', desc: 'Earn more credits', maxLevel: 10,
                costs: [2,3,5,8,12,18,25,35,50,70], effect: l => `+${l*25}% credits`, color: '#ffcc00' },
            startingCredits: { name: 'Trust Fund', icon: '🏦', desc: 'Start with credits after rebirth', maxLevel: 5,
                costs: [3,6,12,20,35], effect: l => `+${l*1000} starting CR`, color: '#ffaa44' },
            weaponMastery: { name: 'Weapon Mastery', icon: '👑', desc: 'Start at higher weapon level', maxLevel: 5,
                costs: [2,5,10,20,40], effect: l => `Start Lv.${l+1} weapons`, color: '#ff00ff' },
            enemyRadar: { name: 'Enemy Radar', icon: '📡', desc: 'See enemy HP bars & warnings', maxLevel: 3,
                costs: [2,5,10], effect: l => `Radar Lv.${l}`, color: '#00ff99' },
            cosmicArmor: { name: 'Cosmic Armor', icon: '🌌', desc: 'Reduce all damage taken', maxLevel: 8,
                costs: [2,4,7,11,16,23,32,45], effect: l => `-${l*5}% damage taken`, color: '#8888ff' },
            voidEnergy: { name: 'Void Energy', icon: '🕳️', desc: 'Explosions deal area damage', maxLevel: 5,
                costs: [3,6,12,24,48], effect: l => `${l*15}% area dmg`, color: '#aa00ff' },
            temporalShift: { name: 'Temporal Shift', icon: '⏳', desc: 'Enemies move slower', maxLevel: 5,
                costs: [3,7,15,30,60], effect: l => `-${l*5}% enemy speed`, color: '#00aaff' }
        };

        // ============ ENEMY DEFINITIONS ============
        const ENEMY_TYPES = {
            drone: { name: 'Scout Drone', minRebirth: 0, hp: 2, speed: 2, score: 250,
                color: '#ff4444', size: 18, shootRate: 0, desc: 'Fast-moving scout' },
            fighter: { name: 'Fighter', minRebirth: 0, hp: 4, speed: 1.5, score: 400,
                color: '#ff8800', size: 22, shootRate: 120, desc: 'Armed fighter craft' },
            bomber: { name: 'Bomber', minRebirth: 1, hp: 8, speed: 0.8, score: 600,
                color: '#aa4400', size: 30, shootRate: 180, desc: 'Heavy bomber - Rebirth 1+' },
            stealth: { name: 'Stealth Ship', minRebirth: 1, hp: 3, speed: 2.5, score: 500,
                color: '#334466', size: 20, shootRate: 90, desc: 'Cloaking stealth craft - Rebirth 1+' },
            carrier: { name: 'Carrier', minRebirth: 2, hp: 15, speed: 0.5, score: 1000,
                color: '#660066', size: 35, shootRate: 200, desc: 'Spawns drones - Rebirth 2+' },
            shielded: { name: 'Shielded Elite', minRebirth: 2, hp: 10, speed: 1.2, score: 800,
                color: '#0088ff', size: 25, shootRate: 100, desc: 'Has regenerating shield - Rebirth 2+' },
            assassin: { name: 'Assassin', minRebirth: 3, hp: 5, speed: 3.5, score: 700,
                color: '#ff0088', size: 16, shootRate: 60, desc: 'Extremely fast attacker - Rebirth 3+' },
            titan: { name: 'Titan Dreadnought', minRebirth: 3, hp: 30, speed: 0.3, score: 2000,
                color: '#880000', size: 45, shootRate: 80, desc: 'Massive warship - Rebirth 3+' },
            phaser: { name: 'Phase Shifter', minRebirth: 4, hp: 6, speed: 2, score: 900,
                color: '#00ffaa', size: 20, shootRate: 70, desc: 'Teleports around - Rebirth 4+' },
            overlord: { name: 'Overlord', minRebirth: 5, hp: 50, speed: 0.4, score: 5000,
                color: '#ffcc00', size: 50, shootRate: 40, desc: 'Ultimate enemy - Rebirth 5+' }
        };

        // ============ SHIP SKINS ============
        const SHIP_SKINS = [
            { name: 'Standard', cost: 0, bodyColor: '#1e3c96', wingColor: '#142d78', accentColor: '#3366dd', cockpitColor: '#00ffff', engineColor: '#ff6600' },
            { name: 'Crimson Hawk', cost: 3000, bodyColor: '#8b0000', wingColor: '#660000', accentColor: '#ff3333', cockpitColor: '#ff9900', engineColor: '#ff0000' },
            { name: 'Ghost', cost: 5000, bodyColor: '#2a2a3a', wingColor: '#1a1a2a', accentColor: '#8888aa', cockpitColor: '#ffffff', engineColor: '#9999ff' },
            { name: 'Solar Flare', cost: 8000, bodyColor: '#cc6600', wingColor: '#994400', accentColor: '#ffaa00', cockpitColor: '#ffff00', engineColor: '#ffcc00' },
            { name: 'Neon Viper', cost: 15000, bodyColor: '#003322', wingColor: '#002211', accentColor: '#00ff88', cockpitColor: '#00ffcc', engineColor: '#00ff44' },
            { name: 'Void Walker', cost: 25000, bodyColor: '#1a0033', wingColor: '#0d001a', accentColor: '#aa44ff', cockpitColor: '#dd88ff', engineColor: '#8800ff' },
            { name: 'Golden Phoenix', cost: 50000, bodyColor: '#665500', wingColor: '#443300', accentColor: '#ffcc00', cockpitColor: '#ffffff', engineColor: '#ffaa00' }
        ];

        // ============ STATE ============
        let state = 'menu';
        let score = 0, credits, highScore;
        credits = saveData.credits;
        highScore = saveData.highScore;
        let wave = 1, combo = 0, comboTimer = 0, waveAnnounceTimer = 0;
        let asteroidsDestroyed = 0, asteroidsToDestroy = 10;
        let gameTime = 0, spawnTimer = 0, enemySpawnTimer = 0;
        let isNewHigh = false;
        let screenShakeX = 0, screenShakeY = 0, screenShakeIntensity = 0, screenShakeDuration = 0;
        let sessionAsteroidsDestroyed = 0, sessionEnemiesDestroyed = 0, creditsEarned = 0;
        let pauseMenuSelection = 0;
        let pauseMenuItems = ['Resume', 'Settings', 'Quit to Menu'];
        let prevState = 'menu';
        let menuSelection = 0;
        let menuItems = [
            { label: 'LAUNCH MISSION', icon: '🚀', action: 'play' },
            { label: 'UPGRADE HANGAR', icon: '🔧', action: 'upgrades' },
            { label: 'REBIRTH', icon: '🌟', action: 'rebirth' },
            { label: 'SHIP SKINS', icon: '🎨', action: 'hangar' },
            { label: 'PILOT STATS', icon: '📊', action: 'stats' },
            { label: 'SETTINGS', icon: '⚙️', action: 'settings' }
        ];
        let upgradeSelection = 0, upgradeCategory = 0;
        let upgradeCategories = ['all', 'offense', 'defense', 'utility'];
        let upgradeCategoryNames = ['ALL', 'OFFENSE', 'DEFENSE', 'UTILITY'];
        let skinSelection = 0, settingsSelection = 0;
        let settingsItems = ['Master Volume', 'SFX Volume', 'Screen Shake', 'Show FPS', 'Particle Density', 'Back'];
        let rebirthSelection = 0, rebirthTab = 0; // 0=info, 1=upgrades
        let rebirthUpgradeSelection = 0;
        let notificationText = '', notificationTimer = 0, notificationColor = '#33ff66';
        let menuShipBob = 0;
        let missileTimer = 0;
        let scrollOffset = 0;

        function showNotification(text, color = '#33ff66') {
            notificationText = text; notificationTimer = 120; notificationColor = color;
        }

        function getUpg(key) { return saveData.upgrades[key] || 0; }
        function getRUpg(key) { return saveData.rebirthUpgrades[key] || 0; }
        function getSkin() { return SHIP_SKINS[saveData.selectedShipSkin] || SHIP_SKINS[0]; }

        function getFilteredUpgrades() {
            let cat = upgradeCategories[upgradeCategory];
            if (cat === 'all') return Object.keys(UPGRADES);
            return Object.keys(UPGRADES).filter(k => UPGRADES[k].category === cat);
        }

        // ============ REBIRTH SYSTEM ============
        function getRebirthCost() {
            return Math.floor(50000 * Math.pow(2.5, saveData.rebirths));
        }

        function getRebirthTokensReward() {
            return Math.floor(3 + saveData.rebirths * 2 + saveData.highScore / 10000);
        }

        function canRebirth() {
            return credits >= getRebirthCost();
        }

        function doRebirth() {
            if (!canRebirth()) return false;
            let tokens = getRebirthTokensReward();
            saveData.rebirths++;
            saveData.rebirthTokens += tokens;
            saveData.totalRebirthTokensEarned += tokens;
            // Reset regular upgrades and credits
            for (let k in saveData.upgrades) saveData.upgrades[k] = 0;
            let startCredits = getRUpg('startingCredits') * 1000;
            saveData.credits = startCredits;
            credits = startCredits;
            saveSave();
            return true;
        }

        function purchaseRebirthUpgrade(key) {
            let upg = REBIRTH_UPGRADES[key];
            let lvl = saveData.rebirthUpgrades[key];
            if (lvl >= upg.maxLevel) { showNotification('MAX LEVEL!', '#ffaa00'); playSound('error'); return; }
            let cost = upg.costs[lvl];
            if (saveData.rebirthTokens >= cost) {
                saveData.rebirthTokens -= cost;
                saveData.rebirthUpgrades[key]++;
                saveSave();
                showNotification(`${upg.name} → Level ${saveData.rebirthUpgrades[key]}!`, '#ff88ff');
                playSound('purchase');
            } else {
                showNotification('Not enough Rebirth Tokens!', '#ff3333');
                playSound('error');
            }
        }

        function getAvailableEnemies() {
            let rb = saveData.rebirths;
            return Object.keys(ENEMY_TYPES).filter(k => ENEMY_TYPES[k].minRebirth <= rb);
        }

        // ============ INPUT ============
        const keys = {};
        document.addEventListener('keydown', (e) => {
            keys[e.key] = true; keys[e.code] = true;
            if (['ArrowUp','ArrowDown','ArrowLeft','ArrowRight',' '].includes(e.key)) e.preventDefault();
            handleInput(e.key, e.code);
        });
        document.addEventListener('keyup', (e) => { keys[e.key] = false; keys[e.code] = false; });

        function handleInput(key, code) {
            if (state === 'menu') {
                initAudio();
                if (key === 'ArrowUp' || key === 'w' || key === 'W') { menuSelection = (menuSelection - 1 + menuItems.length) % menuItems.length; playSound('navigate'); }
                else if (key === 'ArrowDown' || key === 's' || key === 'S') { menuSelection = (menuSelection + 1) % menuItems.length; playSound('navigate'); }
                else if (key === 'Enter' || key === ' ') {
                    let action = menuItems[menuSelection].action;
                    playSound('select');
                    if (action === 'play') { resetGame(); state = 'playing'; saveData.totalGamesPlayed++; saveSave(); }
                    else if (action === 'upgrades') { state = 'upgrades'; upgradeSelection = 0; upgradeCategory = 0; scrollOffset = 0; }
                    else if (action === 'rebirth') { state = 'rebirth'; rebirthTab = 0; rebirthSelection = 0; rebirthUpgradeSelection = 0; }
                    else if (action === 'hangar') { state = 'hangar'; skinSelection = saveData.selectedShipSkin; }
                    else if (action === 'stats') { state = 'stats'; }
                    else if (action === 'settings') { state = 'settings'; settingsSelection = 0; prevState = 'menu'; }
                }
            } else if (state === 'upgrades') {
                let filtered = getFilteredUpgrades();
                if (key === 'ArrowUp' || key === 'w' || key === 'W') { upgradeSelection = (upgradeSelection - 1 + filtered.length) % filtered.length; playSound('navigate'); }
                else if (key === 'ArrowDown' || key === 's' || key === 'S') { upgradeSelection = (upgradeSelection + 1) % filtered.length; playSound('navigate'); }
                else if (key === 'ArrowLeft' || key === 'a' || key === 'A') { upgradeCategory = (upgradeCategory - 1 + upgradeCategories.length) % upgradeCategories.length; upgradeSelection = 0; playSound('navigate'); }
                else if (key === 'ArrowRight' || key === 'd' || key === 'D') { upgradeCategory = (upgradeCategory + 1) % upgradeCategories.length; upgradeSelection = 0; playSound('navigate'); }
                else if (key === 'Enter' || key === ' ') { purchaseUpgrade(filtered[upgradeSelection]); }
                else if (key === 'Escape' || key === 'Backspace') { state = 'menu'; playSound('navigate'); }
            } else if (state === 'rebirth') {
                if (key === 'Tab' || key === 'ArrowLeft' || key === 'ArrowRight' || key === 'a' || key === 'A' || key === 'd' || key === 'D') {
                    if (key === 'ArrowLeft' || key === 'a' || key === 'A') rebirthTab = (rebirthTab - 1 + 3) % 3;
                    else rebirthTab = (rebirthTab + 1) % 3;
                    rebirthUpgradeSelection = 0;
                    playSound('navigate');
                }
                else if (key === 'ArrowUp' || key === 'w' || key === 'W') {
                    if (rebirthTab === 1) {
                        let rKeys = Object.keys(REBIRTH_UPGRADES);
                        rebirthUpgradeSelection = (rebirthUpgradeSelection - 1 + rKeys.length) % rKeys.length;
                    }
                    playSound('navigate');
                }
                else if (key === 'ArrowDown' || key === 's' || key === 'S') {
                    if (rebirthTab === 1) {
                        let rKeys = Object.keys(REBIRTH_UPGRADES);
                        rebirthUpgradeSelection = (rebirthUpgradeSelection + 1) % rKeys.length;
                    }
                    playSound('navigate');
                }
                else if (key === 'Enter' || key === ' ') {
                    if (rebirthTab === 0) {
                        if (canRebirth()) {
                            doRebirth();
                            showNotification(`REBIRTH ${saveData.rebirths}! New enemies unlocked!`, '#ff88ff');
                            playSound('rebirth');
                        } else { showNotification('Not enough credits!', '#ff3333'); playSound('error'); }
                    } else if (rebirthTab === 1) {
                        let rKeys = Object.keys(REBIRTH_UPGRADES);
                        purchaseRebirthUpgrade(rKeys[rebirthUpgradeSelection]);
                    }
                }
                else if (key === 'Escape' || key === 'Backspace') { state = 'menu'; playSound('navigate'); }
            } else if (state === 'hangar') {
                if (key === 'ArrowLeft' || key === 'a' || key === 'A') { skinSelection = (skinSelection - 1 + SHIP_SKINS.length) % SHIP_SKINS.length; playSound('navigate'); }
                else if (key === 'ArrowRight' || key === 'd' || key === 'D') { skinSelection = (skinSelection + 1) % SHIP_SKINS.length; playSound('navigate'); }
                else if (key === 'Enter' || key === ' ') { purchaseSkin(skinSelection); }
                else if (key === 'Escape' || key === 'Backspace') { state = 'menu'; playSound('navigate'); }
            } else if (state === 'settings') {
                if (key === 'ArrowUp' || key === 'w' || key === 'W') { settingsSelection = (settingsSelection - 1 + settingsItems.length) % settingsItems.length; playSound('navigate'); }
                else if (key === 'ArrowDown' || key === 's' || key === 'S') { settingsSelection = (settingsSelection + 1) % settingsItems.length; playSound('navigate'); }
                else if (key === 'ArrowLeft' || key === 'a' || key === 'A') { adjustSetting(settingsSelection, -1); }
                else if (key === 'ArrowRight' || key === 'd' || key === 'D') { adjustSetting(settingsSelection, 1); }
                else if (key === 'Enter' || key === ' ') {
                    if (settingsItems[settingsSelection] === 'Back') { state = prevState === 'paused' ? 'paused' : 'menu'; playSound('navigate'); }
                    else adjustSetting(settingsSelection, 1);
                }
                else if (key === 'Escape' || key === 'Backspace') { state = prevState === 'paused' ? 'paused' : 'menu'; playSound('navigate'); }
            } else if (state === 'stats') {
                if (key === 'Escape' || key === 'Backspace' || key === 'Enter') { state = 'menu'; playSound('navigate'); }
            } else if (state === 'playing') {
                if (key === 'Escape' || key === 'p' || key === 'P') { state = 'paused'; pauseMenuSelection = 0; playSound('navigate'); }
            } else if (state === 'paused') {
                if (key === 'Escape' || key === 'p' || key === 'P') { state = 'playing'; playSound('navigate'); }
                else if (key === 'ArrowUp' || key === 'w' || key === 'W') { pauseMenuSelection = (pauseMenuSelection - 1 + pauseMenuItems.length) % pauseMenuItems.length; playSound('navigate'); }
                else if (key === 'ArrowDown' || key === 's' || key === 'S') { pauseMenuSelection = (pauseMenuSelection + 1) % pauseMenuItems.length; playSound('navigate'); }
                else if (key === 'Enter' || key === ' ') {
                    playSound('select');
                    if (pauseMenuItems[pauseMenuSelection] === 'Resume') state = 'playing';
                    else if (pauseMenuItems[pauseMenuSelection] === 'Settings') { prevState = 'paused'; state = 'settings'; settingsSelection = 0; }
                    else { endGame(false); state = 'menu'; }
                }
            } else if (state === 'gameover') {
                if (key === 'Enter') { resetGame(); state = 'playing'; saveData.totalGamesPlayed++; saveSave(); }
                else if (key === 'Escape') state = 'menu';
            }
        }

        function purchaseUpgrade(key) {
            if (!key) return;
            let upg = UPGRADES[key]; let lvl = saveData.upgrades[key];
            if (lvl >= upg.maxLevel) { showNotification('MAX LEVEL!', '#ffaa00'); playSound('error'); return; }
            let cost = upg.costs[lvl];
            if (credits >= cost) {
                credits -= cost; saveData.upgrades[key]++; saveData.credits = credits; saveSave();
                showNotification(`${upg.name} → Level ${saveData.upgrades[key]}!`, '#33ff66');
                playSound('purchase');
            } else { showNotification('Not enough credits!', '#ff3333'); playSound('error'); }
        }

        function purchaseSkin(i) {
            if (saveData.unlockedSkins[i]) {
                saveData.selectedShipSkin = i; saveSave();
                showNotification(`${SHIP_SKINS[i].name} equipped!`, '#00ffff'); playSound('select');
            } else {
                if (credits >= SHIP_SKINS[i].cost) {
                    credits -= SHIP_SKINS[i].cost; saveData.unlockedSkins[i] = true;
                    saveData.selectedShipSkin = i; saveData.credits = credits; saveSave();
                    showNotification(`${SHIP_SKINS[i].name} unlocked!`, '#33ff66'); playSound('purchase');
                } else { showNotification('Not enough credits!', '#ff3333'); playSound('error'); }
            }
        }

        function adjustSetting(i, dir) {
            let item = settingsItems[i];
            if (item === 'Master Volume') { saveData.settings.masterVolume = Math.max(0, Math.min(1, saveData.settings.masterVolume + dir * 0.1)); masterVolume = saveData.settings.masterVolume; }
            else if (item === 'SFX Volume') { saveData.settings.sfxVolume = Math.max(0, Math.min(1, saveData.settings.sfxVolume + dir * 0.1)); sfxVolume = saveData.settings.sfxVolume; }
            else if (item === 'Screen Shake') saveData.settings.screenShake = !saveData.settings.screenShake;
            else if (item === 'Show FPS') saveData.settings.showFPS = !saveData.settings.showFPS;
            else if (item === 'Particle Density') {
                let v = [0.5,1,1.5,2]; let ci = v.indexOf(saveData.settings.particleDensity);
                if (ci === -1) ci = 1; ci = (ci + dir + v.length) % v.length;
                saveData.settings.particleDensity = v[ci];
            }
            playSound('navigate'); saveSave();
        }

        function endGame(died = true) {
            let creditMult = 1 + getRUpg('creditMultiplier') * 0.25;
            let creditsFromScore = Math.floor(score / 10 * creditMult);
            let waveBonus = Math.floor(wave * 50 * creditMult);
            creditsEarned = creditsFromScore + waveBonus;
            credits += creditsEarned;
            saveData.credits = credits;
            saveData.totalAsteroidsDestroyed += sessionAsteroidsDestroyed;
            saveData.totalEnemiesDestroyed += sessionEnemiesDestroyed;
            if (died) saveData.totalDeaths++;
            if (wave > saveData.highestWave) saveData.highestWave = wave;
            isNewHigh = score > highScore;
            if (score > highScore) { highScore = score; saveData.highScore = highScore; }
            saveSave();
        }

        // ============ STARS ============
        const stars = [];
        for (let i = 0; i < 200; i++) stars.push({
            x: Math.random() * CW, y: Math.random() * CH,
            size: Math.random() * 2.5 + 0.5, brightness: Math.random() * 155 + 100,
            twinkleSpeed: Math.random() * 0.04 + 0.01, phase: Math.random() * Math.PI * 2
        });

        function drawStars(sp = 0.5) {
            stars.forEach(s => {
                s.phase += s.twinkleSpeed;
                let b = Math.max(50, Math.min(255, Math.floor(s.brightness * (0.5 + 0.5 * Math.sin(s.phase)))));
                ctx.fillStyle = `rgb(${b},${b},${b})`;
                ctx.beginPath(); ctx.arc(s.x, s.y, s.size, 0, Math.PI * 2); ctx.fill();
                s.y += sp * s.size;
                if (s.y > CH) { s.y = 0; s.x = Math.random() * CW; }
            });
        }

        // ============ PARTICLES ============
        let particles = [];

        class Particle {
            constructor(x, y, color, vx, vy, size, lifetime, type = 'normal') {
                this.x = x; this.y = y; this.color = color;
                this.vx = vx !== undefined ? vx : (Math.random() - 0.5) * 8;
                this.vy = vy !== undefined ? vy : (Math.random() - 0.5) * 8;
                this.size = size || Math.random() * 4 + 2;
                this.lifetime = lifetime || Math.random() * 25 + 15;
                this.maxLifetime = this.lifetime; this.type = type;
            }
            update() {
                this.x += this.vx; this.y += this.vy; this.lifetime--;
                if (this.type === 'normal') { this.vx *= 0.96; this.vy *= 0.96; this.size *= 0.95; }
                else if (this.type === 'thrust') { this.vy += 0.1; this.size *= 0.92; }
            }
            draw() {
                if (this.lifetime <= 0 || this.size < 0.5) return;
                ctx.globalAlpha = this.lifetime / this.maxLifetime;
                if (this.size > 2) { ctx.shadowBlur = this.size * 3; ctx.shadowColor = this.color; }
                ctx.fillStyle = this.color;
                ctx.beginPath(); ctx.arc(this.x, this.y, Math.max(0.5, this.size), 0, Math.PI * 2); ctx.fill();
                ctx.shadowBlur = 0; ctx.globalAlpha = 1;
            }
        }

        function createExplosion(x, y, color = '#ff9900', count = 20, speed = 5) {
            let d = saveData.settings.particleDensity;
            for (let i = 0; i < Math.floor(count * d); i++) {
                let a = Math.random() * Math.PI * 2, sp = Math.random() * speed + 1;
                let colors = [color, '#ffff00', '#ffffff', '#ff3333'];
                particles.push(new Particle(x, y, colors[Math.floor(Math.random() * colors.length)],
                    Math.cos(a) * sp, Math.sin(a) * sp, Math.random() * 5 + 2, Math.random() * 20 + 15));
            }
        }

        let scorePopups = [];
        function addScorePopup(x, y, text, color = '#ffff00') {
            scorePopups.push({ x, y, text: String(text), color, lifetime: 50, maxLifetime: 50 });
        }

        // ============ SHIP ============
        const ship = {
            x: 400, y: 500, width: 36, height: 44, speed: 5.5,
            health: 100, maxHealth: 100, lives: 3,
            shootCooldown: 0, shootDelay: 12,
            invincible: 0, powerLevel: 1,
            shieldActive: false, shieldTimer: 0, tilt: 0
        };

        function resetShip() {
            ship.x = CW / 2; ship.y = CH - 100;
            ship.maxHealth = 100 + getUpg('hull') * 15 + getRUpg('permanentHP') * 25;
            ship.health = ship.maxHealth;
            ship.lives = 3 + getUpg('extraLives');
            ship.speed = 5.5 + getUpg('speed') * 0.5 + getRUpg('permanentSpeed') * 0.3;
            ship.shootDelay = Math.max(2, 12 - getUpg('fireRate'));
            ship.powerLevel = 1 + getUpg('startPower') + getRUpg('weaponMastery');
            ship.shootCooldown = 0; ship.invincible = 60;
            ship.shieldActive = false; ship.shieldTimer = 0; ship.tilt = 0;
        }

        function updateShip() {
            let ml = false, mr = false;
            if (keys['ArrowLeft'] || keys['a'] || keys['A']) { ship.x -= ship.speed; ml = true; }
            if (keys['ArrowRight'] || keys['d'] || keys['D']) { ship.x += ship.speed; mr = true; }
            if (keys['ArrowUp'] || keys['w'] || keys['W']) ship.y -= ship.speed;
            if (keys['ArrowDown'] || keys['s'] || keys['S']) ship.y += ship.speed;

            if (ml) ship.tilt = Math.max(ship.tilt - 0.5, -5);
            else if (mr) ship.tilt = Math.min(ship.tilt + 0.5, 5);
            else ship.tilt *= 0.85;

            ship.x = Math.max(ship.width / 2, Math.min(CW - ship.width / 2, ship.x));
            ship.y = Math.max(ship.height / 2, Math.min(CH - ship.height / 2, ship.y));

            if (ship.shootCooldown > 0) ship.shootCooldown--;
            if (ship.invincible > 0) ship.invincible--;
            if (ship.shieldTimer > 0) { ship.shieldTimer--; ship.shieldActive = true; }
            else ship.shieldActive = false;

            let skin = getSkin();
            if (Math.random() < 0.7) {
                let colors = [skin.engineColor, '#ffcc00', '#ff3300'];
                particles.push(new Particle(ship.x + (Math.random() - 0.5) * 12, ship.y + ship.height / 2 + 5,
                    colors[Math.floor(Math.random() * colors.length)],
                    (Math.random() - 0.5) * 1.5, Math.random() * 4 + 2,
                    Math.random() * 4 + 2, Math.random() * 15 + 10, 'thrust'));
            }

            // Magnet
            let magRange = getUpg('magnetRange') > 0 ? (50 + getUpg('magnetRange') * 30) : 0;
            if (magRange > 0) {
                powerUps.forEach(pu => {
                    let dx = ship.x - pu.x, dy = ship.y - pu.y;
                    let dist = Math.sqrt(dx * dx + dy * dy);
                    if (dist < magRange && dist > 5) {
                        let pull = 3 * (1 - dist / magRange);
                        pu.x += (dx / dist) * pull; pu.y += (dy / dist) * pull;
                    }
                });
            }

            // Homing missiles
            let hmLvl = getUpg('homingMissiles');
            if (hmLvl > 0) {
                missileTimer++;
                if (missileTimer >= 180) { // Every 3 seconds
                    missileTimer = 0;
                    let targets = [...asteroids, ...enemies].filter(e => e.alive !== false);
                    for (let m = 0; m < hmLvl && targets.length > 0; m++) {
                        let tgt = targets[Math.floor(Math.random() * targets.length)];
                        missiles.push(new Missile(ship.x, ship.y - 10, tgt));
                    }
                    if (targets.length > 0) playSound('shoot');
                }
            }
        }

        function drawShipAt(x, y, scale, skinO) {
            let drawX = x || ship.x, drawY = y || ship.y;
            let s = scale || 1;
            let skin = skinO || getSkin();
            if (!x && ship.invincible > 0 && Math.floor(ship.invincible / 3) % 2 === 0) return;

            ctx.save(); ctx.translate(drawX, drawY); ctx.scale(s, s);

            let grd = ctx.createRadialGradient(0, 0, 5, 0, 0, 40);
            grd.addColorStop(0, 'rgba(0,100,255,0.15)');
            grd.addColorStop(1, 'rgba(0,100,255,0)');
            ctx.fillStyle = grd;
            ctx.beginPath(); ctx.arc(0, 0, 40, 0, Math.PI * 2); ctx.fill();

            let tilt = x ? 0 : ship.tilt;

            ctx.fillStyle = skin.bodyColor; ctx.strokeStyle = skin.accentColor; ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.moveTo(tilt * 2, -ship.height / 2);
            ctx.lineTo(-ship.width / 2, ship.height / 2 - 5);
            ctx.lineTo(-ship.width / 4, ship.height / 2 + 5);
            ctx.lineTo(ship.width / 4, ship.height / 2 + 5);
            ctx.lineTo(ship.width / 2, ship.height / 2 - 5);
            ctx.closePath(); ctx.fill(); ctx.stroke();

            ctx.fillStyle = skin.cockpitColor;
            ctx.beginPath(); ctx.moveTo(tilt, -ship.height / 2 + 10); ctx.lineTo(-8, -5); ctx.lineTo(8, -5); ctx.closePath();
            ctx.fill(); ctx.strokeStyle = '#ffffff'; ctx.lineWidth = 1; ctx.stroke();

            ctx.fillStyle = skin.wingColor; ctx.strokeStyle = skin.accentColor; ctx.lineWidth = 2;
            ctx.beginPath(); ctx.moveTo(-ship.width / 2, ship.height / 2 - 5); ctx.lineTo(-ship.width / 2 - 15, ship.height / 2 + 10);
            ctx.lineTo(-ship.width / 4, ship.height / 4); ctx.closePath(); ctx.fill(); ctx.stroke();
            ctx.beginPath(); ctx.moveTo(ship.width / 2, ship.height / 2 - 5); ctx.lineTo(ship.width / 2 + 15, ship.height / 2 + 10);
            ctx.lineTo(ship.width / 4, ship.height / 4); ctx.closePath(); ctx.fill(); ctx.stroke();

            if (!x) {
                let es = 4 + Math.random() * 4;
                ctx.fillStyle = skin.engineColor; ctx.beginPath(); ctx.arc(0, ship.height / 2 + 5, es, 0, Math.PI * 2); ctx.fill();
                ctx.fillStyle = '#ffffff'; ctx.beginPath(); ctx.arc(0, ship.height / 2 + 5, es / 2, 0, Math.PI * 2); ctx.fill();
            }

            if (!x && ship.shieldActive) {
                let al = 0.3 + 0.15 * Math.sin(Date.now() * 0.01);
                ctx.strokeStyle = `rgba(0,200,255,${al})`; ctx.lineWidth = 3;
                ctx.beginPath(); ctx.arc(0, 0, ship.width * 1.2, 0, Math.PI * 2); ctx.stroke();
            }
            ctx.restore();
        }

        // ============ LASERS ============
        let lasers = [];

        class Laser {
            constructor(x, y, angle = 0, isCrit = false, dmg = 1) {
                this.x = x; this.y = y; this.speed = 14; this.angle = angle;
                this.vx = Math.sin(angle * Math.PI / 180) * this.speed;
                this.vy = -Math.cos(angle * Math.PI / 180) * this.speed;
                this.alive = true; this.trail = [];
                this.isCrit = isCrit; this.damage = dmg * (isCrit ? 2 : 1);
                this.pierceLeft = getUpg('laserPierce');
                this.hitTargets = new Set();
            }
            update() {
                this.trail.push({ x: this.x, y: this.y });
                if (this.trail.length > 5) this.trail.shift();
                this.x += this.vx; this.y += this.vy;
                if (this.y < -20 || this.y > CH + 20 || this.x < -20 || this.x > CW + 20) this.alive = false;
            }
            draw() {
                let lc = this.isCrit ? '#ff3333' : '#39ff14';
                this.trail.forEach((p, i) => {
                    let a = (i + 1) / this.trail.length * 0.5;
                    ctx.fillStyle = this.isCrit ? `rgba(255,50,50,${a})` : `rgba(57,255,20,${a})`;
                    ctx.beginPath(); ctx.arc(p.x, p.y, 2, 0, Math.PI * 2); ctx.fill();
                });
                ctx.shadowBlur = 15; ctx.shadowColor = lc;
                let ex = this.x - this.vx * 1.5, ey = this.y - this.vy * 1.5;
                ctx.strokeStyle = lc; ctx.lineWidth = this.isCrit ? 5 : 4;
                ctx.beginPath(); ctx.moveTo(this.x, this.y); ctx.lineTo(ex, ey); ctx.stroke();
                ctx.strokeStyle = '#ffffff'; ctx.lineWidth = 2;
                ctx.beginPath(); ctx.moveTo(this.x, this.y); ctx.lineTo(ex, ey); ctx.stroke();
                ctx.fillStyle = '#ffffff'; ctx.beginPath(); ctx.arc(this.x, this.y, this.isCrit ? 4 : 3, 0, Math.PI * 2); ctx.fill();
                ctx.shadowBlur = 0;
            }
        }

        // ============ MISSILES ============
        let missiles = [];

        class Missile {
            constructor(x, y, target) {
                this.x = x; this.y = y; this.target = target;
                this.speed = 6; this.alive = true; this.trail = [];
                this.angle = -Math.PI / 2;
                this.damage = 3 + getRUpg('permanentDamage');
                this.lifetime = 300;
            }
            update() {
                this.trail.push({ x: this.x, y: this.y });
                if (this.trail.length > 8) this.trail.shift();
                this.lifetime--;
                if (this.lifetime <= 0) { this.alive = false; return; }

                if (this.target && (this.target.alive !== false)) {
                    let dx = this.target.x - this.x, dy = this.target.y - this.y;
                    let targetAngle = Math.atan2(dy, dx);
                    let diff = targetAngle - this.angle;
                    while (diff > Math.PI) diff -= Math.PI * 2;
                    while (diff < -Math.PI) diff += Math.PI * 2;
                    this.angle += Math.max(-0.08, Math.min(0.08, diff));
                }
                this.x += Math.cos(this.angle) * this.speed;
                this.y += Math.sin(this.angle) * this.speed;
                if (this.x < -20 || this.x > CW + 20 || this.y < -20 || this.y > CH + 20) this.alive = false;

                if (Math.random() < 0.5) particles.push(new Particle(this.x, this.y, '#ff4400',
                    (Math.random() - 0.5) * 2, (Math.random() - 0.5) * 2, 2, 10, 'thrust'));
            }
            draw() {
                this.trail.forEach((p, i) => {
                    let a = (i + 1) / this.trail.length * 0.4;
                    ctx.fillStyle = `rgba(255,68,0,${a})`;
                    ctx.beginPath(); ctx.arc(p.x, p.y, 2, 0, Math.PI * 2); ctx.fill();
                });
                ctx.save(); ctx.translate(this.x, this.y); ctx.rotate(this.angle + Math.PI / 2);
                ctx.fillStyle = '#ff6600'; ctx.beginPath();
                ctx.moveTo(0, -8); ctx.lineTo(-4, 6); ctx.lineTo(4, 6); ctx.closePath(); ctx.fill();
                ctx.fillStyle = '#ffaa00'; ctx.beginPath();
                ctx.moveTo(0, -5); ctx.lineTo(-2, 3); ctx.lineTo(2, 3); ctx.closePath(); ctx.fill();
                ctx.restore();
            }
        }

        function shipShoot() {
            if (ship.shootCooldown <= 0) {
                ship.shootCooldown = ship.shootDelay;
                let critChance = getUpg('critChance') * 0.05;
                let isCrit = Math.random() < critChance;
                let baseDmg = 1 + getUpg('laserDamage') + getRUpg('permanentDamage');
                let power = ship.powerLevel + getUpg('multiShot');
                let spreadBase = 10 + getUpg('spreadShot') * 8;
                let fireCount = getFireCount(power);
                let rearLvl = getUpg('rearGun');

                if (fireCount === 1) {
                    lasers.push(new Laser(ship.x, ship.y - ship.height / 2, 0, isCrit, baseDmg));
                } else {
                    let half = (fireCount - 1) / 2;
                    for (let i = 0; i < fireCount; i++) {
                        let angle = (i - half) * (spreadBase / Math.max(1, fireCount - 1));
                        let offsetX = (i - half) * 5;
                        lasers.push(new Laser(ship.x + offsetX, ship.y - ship.height / 2 + Math.abs(i - half) * 3,
                            angle, isCrit, baseDmg));
                    }
                }

                // Rear guns
                for (let r = 0; r < rearLvl; r++) {
                    let rAngle = 180 + (r - (rearLvl - 1) / 2) * 15;
                    lasers.push(new Laser(ship.x + (r - (rearLvl - 1) / 2) * 8, ship.y + ship.height / 2,
                        rAngle, isCrit, baseDmg));
                }

                playSound('shoot');
            }
        }

        // ============ ASTEROIDS ============
        let asteroids = [];

        class Asteroid {
            constructor(x, y, size) {
                this.size = size || ['large', 'medium', 'small'][Math.floor(Math.random() * 3)];
                let sm = { large: 40, medium: 25, small: 15 };
                this.radius = sm[this.size];
                this.x = x !== undefined ? x : Math.random() * (CW - 80) + 40;
                this.y = y !== undefined ? y : -this.radius - Math.random() * 80;
                let spdMult = 1 - getRUpg('temporalShift') * 0.05;
                this.vx = (Math.random() - 0.5) * 4 * spdMult;
                this.vy = (Math.random() * 2 + 1) * spdMult;
                this.rotation = Math.random() * 360;
                this.rotSpeed = (Math.random() - 0.5) * 4;
                let hm = { large: 3, medium: 2, small: 1 };
                this.health = hm[this.size]; this.maxHealth = this.health;
                this.alive = true; this.hitFlash = 0;
                this.color = ['#8b7765', '#a08c78', '#786450', '#b4a08c'][Math.floor(Math.random() * 4)];
                this.vertices = [];
                let nv = Math.floor(Math.random() * 5) + 7;
                for (let i = 0; i < nv; i++) {
                    this.vertices.push({ angle: (i / nv) * Math.PI * 2, dist: this.radius * (0.7 + Math.random() * 0.3) });
                }
                this.scoreValue = { large: 100, medium: 200, small: 300 }[this.size];
                this.id = Math.random();
                this.isEnemy = false;
            }
            update() {
                this.x += this.vx; this.y += this.vy; this.rotation += this.rotSpeed;
                if (this.hitFlash > 0) this.hitFlash--;
                if (this.x < -this.radius) this.x = CW + this.radius;
                else if (this.x > CW + this.radius) this.x = -this.radius;
                if (this.y > CH + this.radius * 2) this.alive = false;
            }
            draw() {
                ctx.save(); ctx.translate(this.x, this.y);
                let pts = this.vertices.map(v => {
                    let a = v.angle + this.rotation * Math.PI / 180;
                    return { x: Math.cos(a) * v.dist, y: Math.sin(a) * v.dist };
                });
                ctx.fillStyle = 'rgba(20,20,30,0.6)'; ctx.beginPath(); ctx.moveTo(pts[0].x + 3, pts[0].y + 3);
                for (let i = 1; i < pts.length; i++) ctx.lineTo(pts[i].x + 3, pts[i].y + 3); ctx.closePath(); ctx.fill();
                ctx.fillStyle = this.hitFlash > 0 ? '#ffffff' : this.color;
                ctx.strokeStyle = this.hitFlash > 0 ? '#ffffff' : '#5a4a3a'; ctx.lineWidth = 2;
                ctx.beginPath(); ctx.moveTo(pts[0].x, pts[0].y);
                for (let i = 1; i < pts.length; i++) ctx.lineTo(pts[i].x, pts[i].y); ctx.closePath(); ctx.fill(); ctx.stroke();
                ctx.restore();
                if (this.size !== 'small' && this.health < this.maxHealth) {
                    let bw = this.radius * 2, bx = this.x - bw / 2, by = this.y - this.radius - 10;
                    ctx.fillStyle = '#ff3333'; ctx.fillRect(bx, by, bw, 4);
                    ctx.fillStyle = '#33ff33'; ctx.fillRect(bx, by, bw * (this.health / this.maxHealth), 4);
                }
            }
            hit(dmg = 1) { this.health -= dmg; this.hitFlash = 5; return this.health <= 0; }
            split() {
                let arr = [];
                if (this.size === 'large') for (let i = 0; i < 2; i++) { let a = new Asteroid(this.x + (Math.random() - 0.5) * 30, this.y, 'medium'); a.vx = (Math.random() - 0.5) * 5; arr.push(a); }
                else if (this.size === 'medium') for (let i = 0; i < 2; i++) { let a = new Asteroid(this.x + (Math.random() - 0.5) * 20, this.y, 'small'); a.vx = (Math.random() - 0.5) * 6; arr.push(a); }
                return arr;
            }
        }

        // ============ ENEMIES ============
        let enemies = [];
        let enemyLasers = [];

        class Enemy {
            constructor(type) {
                let def = ENEMY_TYPES[type];
                this.type = type; this.name = def.name;
                this.x = Math.random() * (CW - 100) + 50; this.y = -40;
                let spdMult = 1 - getRUpg('temporalShift') * 0.05;
                this.baseSpeed = def.speed * spdMult;
                this.vx = (Math.random() - 0.5) * 2; this.vy = this.baseSpeed;
                this.maxHealth = def.hp; this.health = def.hp;
                this.size = def.size; this.radius = def.size;
                this.color = def.color; this.score = def.score;
                this.shootRate = def.shootRate; this.shootTimer = Math.random() * this.shootRate;
                this.alive = true; this.hitFlash = 0;
                this.id = Math.random(); this.isEnemy = true;
                this.moveTimer = 0; this.movePhase = Math.random() * Math.PI * 2;

                // Special properties
                this.shieldHP = type === 'shielded' ? 5 : 0;
                this.maxShieldHP = this.shieldHP;
                this.shieldRegenTimer = 0;
                this.cloakAlpha = type === 'stealth' ? 0.3 : 1;
                this.phaseTimer = type === 'phaser' ? 0 : -1;
                this.carrierSpawnTimer = type === 'carrier' ? 300 : -1;
            }
            update() {
                this.moveTimer++;
                this.movePhase += 0.02;
                if (this.hitFlash > 0) this.hitFlash--;

                // Movement patterns
                if (this.type === 'assassin') {
                    let dx = ship.x - this.x, dy = ship.y - this.y;
                    let dist = Math.sqrt(dx * dx + dy * dy);
                    if (dist > 5) { this.vx = (dx / dist) * this.baseSpeed; this.vy = (dy / dist) * this.baseSpeed; }
                } else if (this.type === 'phaser') {
                    this.phaseTimer++;
                    if (this.phaseTimer > 120) {
                        this.phaseTimer = 0;
                        this.x = Math.random() * (CW - 80) + 40;
                        this.y = Math.random() * (CH / 2 - 40) + 40;
                        createExplosion(this.x, this.y, '#00ffaa', 10, 3);
                    }
                    this.vx = Math.sin(this.movePhase) * 1.5;
                } else {
                    this.vx = Math.sin(this.movePhase) * 2;
                    if (this.y > CH / 3) this.vy = Math.sin(this.moveTimer * 0.01) * 0.5;
                }

                this.x += this.vx; this.y += this.vy;
                this.x = Math.max(this.size, Math.min(CW - this.size, this.x));
                if (this.y > CH + 50) this.alive = false;

                // Stealth cloaking
                if (this.type === 'stealth') {
                    let dist = Math.sqrt((ship.x - this.x) ** 2 + (ship.y - this.y) ** 2);
                    this.cloakAlpha = dist < 200 ? Math.min(1, this.cloakAlpha + 0.02) : Math.max(0.15, this.cloakAlpha - 0.01);
                }

                // Shield regen
                if (this.type === 'shielded' && this.shieldHP < this.maxShieldHP) {
                    this.shieldRegenTimer++;
                    if (this.shieldRegenTimer > 180) { this.shieldHP = Math.min(this.maxShieldHP, this.shieldHP + 1); this.shieldRegenTimer = 0; }
                }

                // Carrier spawn
                if (this.carrierSpawnTimer >= 0) {
                    this.carrierSpawnTimer--;
                    if (this.carrierSpawnTimer <= 0) {
                        this.carrierSpawnTimer = 300;
                        let drone = new Enemy('drone');
                        drone.x = this.x; drone.y = this.y + 20;
                        enemies.push(drone);
                    }
                }

                // Shooting
                if (this.shootRate > 0) {
                    this.shootTimer++;
                    if (this.shootTimer >= this.shootRate) {
                        this.shootTimer = 0;
                        let dx = ship.x - this.x, dy = ship.y - this.y;
                        let dist = Math.sqrt(dx * dx + dy * dy);
                        if (dist > 5 && this.y > 0 && this.y < CH - 50) {
                            enemyLasers.push({
                                x: this.x, y: this.y + this.size / 2,
                                vx: (dx / dist) * 5, vy: (dy / dist) * 5,
                                alive: true, color: this.color
                            });
                            playSound('enemyShoot');
                        }
                    }
                }
            }
            draw() {
                ctx.save();
                ctx.globalAlpha = this.type === 'stealth' ? this.cloakAlpha : 1;
                ctx.translate(this.x, this.y);

                // Body - hexagonal shape
                let sides = this.type === 'titan' || this.type === 'overlord' ? 8 : 6;
                ctx.fillStyle = this.hitFlash > 0 ? '#ffffff' : this.color;
                ctx.strokeStyle = '#ffffff'; ctx.lineWidth = 2;
                ctx.beginPath();
                for (let i = 0; i < sides; i++) {
                    let a = (i / sides) * Math.PI * 2 - Math.PI / 2;
                    let r = this.size;
                    if (i % 2 === 0) r *= 0.85;
                    ctx.lineTo(Math.cos(a) * r, Math.sin(a) * r);
                }
                ctx.closePath(); ctx.fill(); ctx.stroke();

                // Inner detail
                ctx.fillStyle = 'rgba(255,255,255,0.2)';
                ctx.beginPath();
                for (let i = 0; i < sides; i++) {
                    let a = (i / sides) * Math.PI * 2 - Math.PI / 2;
                    ctx.lineTo(Math.cos(a) * this.size * 0.5, Math.sin(a) * this.size * 0.5);
                }
                ctx.closePath(); ctx.fill();

                // Eye/cockpit
                ctx.fillStyle = '#ff0000'; ctx.beginPath();
                ctx.arc(0, -this.size * 0.2, this.size * 0.2, 0, Math.PI * 2); ctx.fill();
                ctx.fillStyle = '#ffffff'; ctx.beginPath();
                ctx.arc(0, -this.size * 0.2, this.size * 0.1, 0, Math.PI * 2); ctx.fill();

                // Shield visual
                if (this.shieldHP > 0) {
                    let sa = 0.3 + 0.15 * Math.sin(Date.now() * 0.005);
                    ctx.strokeStyle = `rgba(0,136,255,${sa})`; ctx.lineWidth = 3;
                    ctx.beginPath(); ctx.arc(0, 0, this.size + 8, 0, Math.PI * 2); ctx.stroke();
                }

                ctx.restore();
                ctx.globalAlpha = 1;

                // HP bar
                if (this.health < this.maxHealth || getRUpg('enemyRadar') > 0) {
                    let bw = this.size * 2, bx = this.x - bw / 2, by = this.y - this.size - 12;
                    ctx.fillStyle = '#ff3333'; ctx.fillRect(bx, by, bw, 4);
                    ctx.fillStyle = '#33ff33'; ctx.fillRect(bx, by, bw * (this.health / this.maxHealth), 4);
                    if (this.shieldHP > 0) {
                        ctx.fillStyle = '#0088ff'; ctx.fillRect(bx, by - 6, bw * (this.shieldHP / this.maxShieldHP), 3);
                    }
                }

                // Name tag (radar upgrade)
                if (getRUpg('enemyRadar') >= 2) {
                    ctx.fillStyle = 'rgba(255,255,255,0.6)'; ctx.font = '10px Arial';
                    ctx.textAlign = 'center'; ctx.fillText(this.name, this.x, this.y - this.size - 16);
                }
            }
            hit(dmg = 1) {
                if (this.shieldHP > 0) {
                    this.shieldHP -= dmg;
                    this.shieldRegenTimer = 0;
                    if (this.shieldHP < 0) { dmg = -this.shieldHP; this.shieldHP = 0; }
                    else { this.hitFlash = 5; return false; }
                }
                this.health -= dmg; this.hitFlash = 5;
                return this.health <= 0;
            }
        }

        // ============ POWER-UPS ============
        let powerUps = [];

        class PowerUp {
            constructor(x, y) {
                this.x = x; this.y = y; this.vy = 2; this.radius = 15; this.alive = true;
                this.type = ['power', 'health', 'shield', 'bomb', 'credit'][Math.floor(Math.random() * 5)];
                this.colorMap = { power: '#ff1493', health: '#33ff66', shield: '#00ffff', bomb: '#ff8800', credit: '#ffcc00' };
                this.iconMap = { power: 'P', health: '+', shield: 'S', bomb: 'B', credit: '$' };
                this.pulse = 0;
            }
            update() { this.y += this.vy; this.pulse += 0.1; if (this.y > CH + 20) this.alive = false; }
            draw() {
                let c = this.colorMap[this.type]; let ps = 2 * Math.sin(this.pulse);
                ctx.shadowBlur = 20; ctx.shadowColor = c;
                ctx.fillStyle = c; ctx.beginPath(); ctx.arc(this.x, this.y, this.radius + ps, 0, Math.PI * 2); ctx.fill();
                ctx.strokeStyle = '#ffffff'; ctx.lineWidth = 2; ctx.stroke(); ctx.shadowBlur = 0;
                ctx.fillStyle = '#ffffff'; ctx.font = 'bold 16px Arial'; ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
                ctx.fillText(this.iconMap[this.type], this.x, this.y);
            }
        }

        // ============ HELPERS ============
        function startShake(i = 5, d = 10) { if (!saveData.settings.screenShake) return; screenShakeIntensity = i; screenShakeDuration = d; }
        function updateShake() {
            if (screenShakeDuration > 0) { screenShakeDuration--; screenShakeX = (Math.random() - 0.5) * screenShakeIntensity * 2; screenShakeY = (Math.random() - 0.5) * screenShakeIntensity * 2; screenShakeIntensity *= 0.9; }
            else { screenShakeX = 0; screenShakeY = 0; }
        }

        function roundRect(ctx, x, y, w, h, r, fill, stroke) {
            ctx.beginPath(); ctx.moveTo(x + r, y); ctx.arcTo(x + w, y, x + w, y + h, r);
            ctx.arcTo(x + w, y + h, x, y + h, r); ctx.arcTo(x, y + h, x, y, r);
            ctx.arcTo(x, y, x + w, y, r); ctx.closePath();
            if (fill) ctx.fill(); if (stroke) ctx.stroke();
        }

        function drawPanel(x, y, w, h, bc = '#3366ff', ba = 0.85) {
            ctx.fillStyle = `rgba(5,5,30,${ba})`; roundRect(ctx, x, y, w, h, 10, true, false);
            ctx.strokeStyle = bc; ctx.lineWidth = 2; roundRect(ctx, x, y, w, h, 10, false, true);
        }

        function drawProgressBar(x, y, w, h, val, max, c) {
            ctx.fillStyle = '#222244'; roundRect(ctx, x, y, w, h, h / 2, true, false);
            if (val > 0) { ctx.fillStyle = c; roundRect(ctx, x, y, w * Math.min(val / max, 1), h, h / 2, true, false); }
            ctx.strokeStyle = '#555577'; ctx.lineWidth = 1; roundRect(ctx, x, y, w, h, h / 2, false, true);
        }

        function drawCreditsDisplay(x, y) {
            ctx.fillStyle = '#ffcc00'; ctx.font = 'bold 16px Arial'; ctx.textAlign = 'right'; ctx.textBaseline = 'top';
            ctx.shadowBlur = 5; ctx.shadowColor = '#ffcc00';
            ctx.fillText(`💰 ${credits.toLocaleString()} CR`, x, y);
            if (saveData.rebirthTokens > 0) {
                ctx.fillStyle = '#ff88ff'; ctx.shadowColor = '#ff88ff';
                ctx.fillText(`🌟 ${saveData.rebirthTokens} RT`, x, y + 20);
            }
            ctx.shadowBlur = 0;
        }

        function drawNotification() {
            if (notificationTimer <= 0) return; notificationTimer--;
            let a = Math.min(1, notificationTimer / 20); ctx.globalAlpha = a;
            ctx.font = 'bold 18px Arial'; let tw = ctx.measureText(notificationText).width;
            let pw = tw + 60, px = CW / 2 - pw / 2, py = CH - 70;
            ctx.fillStyle = 'rgba(0,0,0,0.85)'; roundRect(ctx, px, py, pw, 40, 8, true, false);
            ctx.strokeStyle = notificationColor; ctx.lineWidth = 2; roundRect(ctx, px, py, pw, 40, 8, false, true);
            ctx.fillStyle = notificationColor; ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
            ctx.fillText(notificationText, CW / 2, py + 20); ctx.globalAlpha = 1;
        }

        function circleCollision(x1, y1, r1, x2, y2, r2) { let dx = x1 - x2, dy = y1 - y2; return Math.sqrt(dx * dx + dy * dy) < r1 + r2; }

        // ============ MENU ============
        function drawMainMenu() {
            ctx.fillStyle = '#05051e'; ctx.fillRect(0, 0, CW, CH); drawStars(0.3);
            menuShipBob += 0.03;
            let t = Date.now() * 0.001;

            // Rebirth stars background
            if (saveData.rebirths > 0) {
                for (let i = 0; i < saveData.rebirths * 3; i++) {
                    let sx = CW / 2 + Math.cos(t * 0.5 + i * 1.1) * (150 + i * 20);
                    let sy = 80 + Math.sin(t * 0.3 + i * 0.8) * 30;
                    ctx.globalAlpha = 0.3; ctx.fillStyle = '#ff88ff';
                    ctx.beginPath(); ctx.arc(sx, sy, 2, 0, Math.PI * 2); ctx.fill();
                    ctx.globalAlpha = 1;
                }
            }

            // Title
            ctx.shadowBlur = 40; ctx.shadowColor = '#0066ff';
            ctx.fillStyle = '#00ccff'; ctx.font = 'bold 56px Arial'; ctx.textAlign = 'center';
            ctx.fillText('ASTEROID', CW / 2, 75);
            ctx.shadowColor = '#00ff66'; ctx.fillStyle = '#39ff14'; ctx.fillText('BLASTER', CW / 2, 135);
            ctx.shadowBlur = 0;

            // Rebirth indicator
            if (saveData.rebirths > 0) {
                ctx.fillStyle = '#ff88ff'; ctx.font = 'bold 16px Arial';
                let rebirthText = '⭐'.repeat(Math.min(saveData.rebirths, 10)) + (saveData.rebirths > 10 ? ` +${saveData.rebirths - 10}` : '');
                ctx.fillText(`Rebirth ${saveData.rebirths}  ${rebirthText}`, CW / 2, 158);
            }

            ctx.fillStyle = '#6688aa'; ctx.font = '12px Arial';
            ctx.fillText('DEFEND THE GALAXY • UPGRADE • REBIRTH • BECOME LEGENDARY', CW / 2, 175);

            // Ship
            let shipY = 215 + Math.sin(menuShipBob) * 8;
            drawShipAt(CW / 2, shipY, 1.2, getSkin());
            let fs = 5 + Math.random() * 5; let skin = getSkin();
            ctx.fillStyle = skin.engineColor; ctx.beginPath();
            ctx.arc(CW / 2, shipY + 32, fs, 0, Math.PI * 2); ctx.fill();

            // Menu items
            let msy = 280, mih = 42, mw = 300, mx = CW / 2 - mw / 2;
            menuItems.forEach((item, i) => {
                let y = msy + i * (mih + 6); let sel = i === menuSelection;
                let colors = ['#0088ff', '#ff6600', '#ff44ff', '#cc33ff', '#33ccff', '#888888'];
                let c = colors[i] || '#3366ff';
                if (sel) {
                    ctx.shadowBlur = 15; ctx.shadowColor = c; ctx.fillStyle = c;
                    roundRect(ctx, mx, y, mw, mih, 10, true, false);
                    ctx.strokeStyle = '#ffffff'; ctx.lineWidth = 2; roundRect(ctx, mx, y, mw, mih, 10, false, true);
                    ctx.shadowBlur = 0;
                    ctx.fillStyle = '#ffffff'; ctx.font = 'bold 18px Arial'; ctx.textAlign = 'left';
                    ctx.fillText('▶', mx - 22, y + mih / 2 + 1);
                    ctx.textAlign = 'center'; ctx.font = 'bold 20px Arial';
                    ctx.fillText(`${item.icon}  ${item.label}`, CW / 2, y + mih / 2 + 1);
                    // Show rebirth token info
                    if (item.action === 'rebirth' && saveData.rebirths > 0) {
                        ctx.fillStyle = '#ffcc00'; ctx.font = '11px Arial'; ctx.textAlign = 'right';
                        ctx.fillText(`🌟 ${saveData.rebirthTokens} tokens`, mx + mw - 5, y + mih - 5);
                    }
                } else {
                    ctx.fillStyle = 'rgba(10,10,40,0.7)'; roundRect(ctx, mx, y, mw, mih, 10, true, false);
                    ctx.strokeStyle = `${c}44`; ctx.lineWidth = 1; roundRect(ctx, mx, y, mw, mih, 10, false, true);
                    ctx.fillStyle = '#8899bb'; ctx.font = '17px Arial'; ctx.textAlign = 'center';
                    ctx.fillText(`${item.icon}  ${item.label}`, CW / 2, y + mih / 2 + 1);
                }
            });

            drawCreditsDisplay(CW - 20, 10);

            if (highScore > 0) {
                ctx.fillStyle = 'rgba(0,0,0,0.5)'; roundRect(ctx, CW / 2 - 120, CH - 42, 240, 30, 8, true, false);
                ctx.fillStyle = '#ffcc00'; ctx.font = 'bold 16px Arial'; ctx.textAlign = 'center';
                ctx.fillText(`🏆 High Score: ${highScore.toLocaleString()}`, CW / 2, CH - 24);
            }

            ctx.fillStyle = '#555577'; ctx.font = '12px Arial'; ctx.textAlign = 'center';
            ctx.fillText('↑↓ Navigate • ENTER Select', CW / 2, CH - 5);
            drawNotification();
        }

        // ============ UPGRADE SCREEN ============
        function drawUpgradeScreen() {
            ctx.fillStyle = '#05051e'; ctx.fillRect(0, 0, CW, CH); drawStars(0.1);

            drawPanel(15, 5, CW - 30, 42, '#ff6600');
            ctx.fillStyle = '#ff8833'; ctx.font = 'bold 24px Arial'; ctx.textAlign = 'center';
            ctx.fillText('🔧 UPGRADE HANGAR', CW / 2, 30);
            drawCreditsDisplay(CW - 30, 10);

            // Category tabs
            let tabW = 100, tabY = 52;
            upgradeCategories.forEach((cat, i) => {
                let tx = 30 + i * (tabW + 8);
                let sel = i === upgradeCategory;
                ctx.fillStyle = sel ? '#334488' : 'rgba(10,10,35,0.7)';
                roundRect(ctx, tx, tabY, tabW, 28, 6, true, false);
                ctx.strokeStyle = sel ? '#6688ff' : '#333355'; ctx.lineWidth = sel ? 2 : 1;
                roundRect(ctx, tx, tabY, tabW, 28, 6, false, true);
                ctx.fillStyle = sel ? '#ffffff' : '#888899'; ctx.font = sel ? 'bold 13px Arial' : '13px Arial';
                ctx.textAlign = 'center'; ctx.fillText(upgradeCategoryNames[i], tx + tabW / 2, tabY + 16);
            });

            let filtered = getFilteredUpgrades();
            let sy = 88, ih = 58, lx = 25, lw = CW - 50;
            let maxVisible = Math.floor((CH - sy - 30) / (ih + 3));

            // Scrolling
            if (upgradeSelection >= scrollOffset + maxVisible) scrollOffset = upgradeSelection - maxVisible + 1;
            if (upgradeSelection < scrollOffset) scrollOffset = upgradeSelection;

            for (let vi = 0; vi < maxVisible && vi + scrollOffset < filtered.length; vi++) {
                let i = vi + scrollOffset;
                let key = filtered[i]; let upg = UPGRADES[key]; let lvl = saveData.upgrades[key];
                let sel = i === upgradeSelection; let y = sy + vi * (ih + 3);

                if (sel) { ctx.shadowBlur = 8; ctx.shadowColor = upg.color; }
                ctx.fillStyle = sel ? 'rgba(30,30,80,0.95)' : 'rgba(10,10,35,0.8)';
                roundRect(ctx, lx, y, lw, ih, 6, true, false);
                ctx.strokeStyle = sel ? upg.color : '#333355'; ctx.lineWidth = sel ? 2 : 1;
                roundRect(ctx, lx, y, lw, ih, 6, false, true); ctx.shadowBlur = 0;

                ctx.fillStyle = upg.color; ctx.font = 'bold 17px Arial'; ctx.textAlign = 'left';
                ctx.fillText(`${upg.icon} ${upg.name}`, lx + 12, y + 20);
                ctx.fillStyle = '#8888aa'; ctx.font = '11px Arial';
                ctx.fillText(upg.desc, lx + 12, y + 36);

                let lvlX = lx + lw - 310;
                ctx.fillStyle = '#aaaacc'; ctx.font = '12px Arial'; ctx.textAlign = 'left';
                ctx.fillText(`Lv.${lvl}/${upg.maxLevel}`, lvlX, y + 16);
                drawProgressBar(lvlX, y + 22, 120, 8, lvl, upg.maxLevel, upg.color);
                ctx.fillStyle = '#ccccee'; ctx.font = '11px Arial'; ctx.fillText(upg.effect(lvl), lvlX, y + 46);

                let cx = lx + lw - 110;
                if (lvl >= upg.maxLevel) {
                    ctx.fillStyle = '#ffcc00'; ctx.font = 'bold 14px Arial'; ctx.textAlign = 'center';
                    ctx.fillText('✓ MAXED', cx + 45, y + ih / 2);
                } else {
                    let cost = upg.costs[lvl]; let afford = credits >= cost;
                    ctx.fillStyle = afford ? '#33ff66' : '#ff4444'; ctx.font = 'bold 13px Arial'; ctx.textAlign = 'center';
                    ctx.fillText(`💰 ${cost.toLocaleString()}`, cx + 45, y + ih / 2 - 5);
                    if (sel) { ctx.fillStyle = afford ? '#aaffaa' : '#ffaaaa'; ctx.font = '10px Arial'; ctx.fillText(afford ? '[ENTER]' : 'Need credits', cx + 45, y + ih / 2 + 12); }
                }
                if (sel) { ctx.fillStyle = upg.color; ctx.font = 'bold 16px Arial'; ctx.textAlign = 'right'; ctx.fillText('▶', lx - 2, y + ih / 2 + 2); }
            }

            // Scroll indicator
            if (filtered.length > maxVisible) {
                ctx.fillStyle = '#555577'; ctx.font = '12px Arial'; ctx.textAlign = 'center';
                ctx.fillText(`${upgradeSelection + 1}/${filtered.length}`, CW - 30, CH / 2);
            }

            ctx.fillStyle = '#555577'; ctx.font = '12px Arial'; ctx.textAlign = 'center';
            ctx.fillText('↑↓ Select • ←→ Category • ENTER Buy • ESC Back', CW / 2, CH - 5);
            drawNotification();
        }

        // ============ REBIRTH SCREEN ============
        function drawRebirthScreen() {
            ctx.fillStyle = '#05051e'; ctx.fillRect(0, 0, CW, CH); drawStars(0.15);

            // Cosmic background effect
            let t = Date.now() * 0.001;
            for (let i = 0; i < 30; i++) {
                let sx = CW / 2 + Math.cos(t * 0.3 + i * 0.5) * (100 + i * 10);
                let sy = CH / 2 + Math.sin(t * 0.4 + i * 0.7) * (80 + i * 8);
                ctx.globalAlpha = 0.15; ctx.fillStyle = `hsl(${(i * 30 + t * 50) % 360}, 80%, 60%)`;
                ctx.beginPath(); ctx.arc(sx, sy, 3, 0, Math.PI * 2); ctx.fill();
            }
            ctx.globalAlpha = 1;

            drawPanel(15, 5, CW - 30, 42, '#ff44ff');
            ctx.fillStyle = '#ff88ff'; ctx.font = 'bold 24px Arial'; ctx.textAlign = 'center';
            ctx.fillText(`🌟 REBIRTH (Current: ${saveData.rebirths})`, CW / 2, 30);
            drawCreditsDisplay(CW - 30, 10);

            // Tabs
            let tabs = ['REBIRTH INFO', 'REBIRTH UPGRADES', 'ENEMY CODEX'];
            let tabW = (CW - 60) / 3;
            tabs.forEach((tab, i) => {
                let tx = 20 + i * (tabW + 5); let sel = i === rebirthTab;
                ctx.fillStyle = sel ? '#553388' : 'rgba(10,10,35,0.7)';
                roundRect(ctx, tx, 52, tabW, 30, 6, true, false);
                ctx.strokeStyle = sel ? '#aa66ff' : '#333355'; ctx.lineWidth = sel ? 2 : 1;
                roundRect(ctx, tx, 52, tabW, 30, 6, false, true);
                ctx.fillStyle = sel ? '#ffffff' : '#888899'; ctx.font = sel ? 'bold 13px Arial' : '13px Arial';
                ctx.textAlign = 'center'; ctx.fillText(tab, tx + tabW / 2, 70);
            });

            if (rebirthTab === 0) {
                // Rebirth info
                let py = 100;
                drawPanel(30, py, CW - 60, CH - py - 30, '#aa44ff');

                ctx.fillStyle = '#ffffff'; ctx.font = 'bold 28px Arial'; ctx.textAlign = 'center';
                ctx.fillText('🌟 REBIRTH SYSTEM', CW / 2, py + 40);

                ctx.fillStyle = '#ccaaee'; ctx.font = '15px Arial';
                let info = [
                    'Rebirthing resets your credits & upgrades,',
                    'but grants powerful permanent Rebirth Tokens!',
                    '',
                    'Each rebirth unlocks NEW ENEMY TYPES',
                    'and access to REBIRTH-ONLY upgrades.',
                    ''
                ];
                info.forEach((line, i) => ctx.fillText(line, CW / 2, py + 75 + i * 22));

                let costY = py + 210;
                ctx.fillStyle = '#ffffff'; ctx.font = 'bold 20px Arial';
                ctx.fillText(`Rebirth Cost: 💰 ${getRebirthCost().toLocaleString()} CR`, CW / 2, costY);
                ctx.fillStyle = credits >= getRebirthCost() ? '#33ff66' : '#ff4444';
                ctx.font = '16px Arial';
                ctx.fillText(`You have: ${credits.toLocaleString()} CR`, CW / 2, costY + 28);

                ctx.fillStyle = '#ffcc00'; ctx.font = 'bold 18px Arial';
                ctx.fillText(`Reward: 🌟 ${getRebirthTokensReward()} Rebirth Tokens`, CW / 2, costY + 60);

                // Rebirth button
                let btnY = costY + 100;
                let canDo = canRebirth();
                ctx.fillStyle = canDo ? '#aa44ff' : '#333344';
                roundRect(ctx, CW / 2 - 120, btnY, 240, 50, 12, true, false);
                ctx.strokeStyle = canDo ? '#ffffff' : '#555555'; ctx.lineWidth = 2;
                roundRect(ctx, CW / 2 - 120, btnY, 240, 50, 12, false, true);
                ctx.fillStyle = canDo ? '#ffffff' : '#666666'; ctx.font = 'bold 22px Arial';
                ctx.fillText(canDo ? '⟐ REBIRTH NOW ⟐' : '🔒 Not Enough CR', CW / 2, btnY + 28);

                // What you'll unlock
                let nextRebirth = saveData.rebirths + 1;
                let newEnemies = Object.keys(ENEMY_TYPES).filter(k => ENEMY_TYPES[k].minRebirth === nextRebirth);
                if (newEnemies.length > 0) {
                    ctx.fillStyle = '#ff8888'; ctx.font = 'bold 15px Arial';
                    ctx.fillText(`Rebirth ${nextRebirth} unlocks:`, CW / 2, btnY + 80);
                    ctx.fillStyle = '#ffaaaa'; ctx.font = '14px Arial';
                    newEnemies.forEach((ek, i) => {
                        ctx.fillText(`⚠️ ${ENEMY_TYPES[ek].name} - ${ENEMY_TYPES[ek].desc}`, CW / 2, btnY + 100 + i * 20);
                    });
                }
            } else if (rebirthTab === 1) {
                // Rebirth upgrades
                let rKeys = Object.keys(REBIRTH_UPGRADES);
                let sy = 92, ih = 54, lx = 25, lw = CW - 50;
                let maxVis = Math.floor((CH - sy - 30) / (ih + 3));

                for (let vi = 0; vi < maxVis && vi < rKeys.length; vi++) {
                    let key = rKeys[vi]; let upg = REBIRTH_UPGRADES[key]; let lvl = saveData.rebirthUpgrades[key];
                    let sel = vi === rebirthUpgradeSelection; let y = sy + vi * (ih + 3);

                    if (sel) { ctx.shadowBlur = 8; ctx.shadowColor = upg.color; }
                    ctx.fillStyle = sel ? 'rgba(40,20,60,0.95)' : 'rgba(15,5,25,0.8)';
                    roundRect(ctx, lx, y, lw, ih, 6, true, false);
                    ctx.strokeStyle = sel ? upg.color : '#442255'; ctx.lineWidth = sel ? 2 : 1;
                    roundRect(ctx, lx, y, lw, ih, 6, false, true); ctx.shadowBlur = 0;

                    ctx.fillStyle = upg.color; ctx.font = 'bold 16px Arial'; ctx.textAlign = 'left';
                    ctx.fillText(`${upg.icon} ${upg.name}`, lx + 12, y + 19);
                    ctx.fillStyle = '#aa88cc'; ctx.font = '11px Arial'; ctx.fillText(upg.desc, lx + 12, y + 35);

                    let lvlX = lx + lw - 310;
                    ctx.fillStyle = '#ccaaee'; ctx.font = '12px Arial'; ctx.textAlign = 'left';
                    ctx.fillText(`Lv.${lvl}/${upg.maxLevel}`, lvlX, y + 16);
                    drawProgressBar(lvlX, y + 22, 120, 8, lvl, upg.maxLevel, upg.color);
                    ctx.fillStyle = '#ddccee'; ctx.font = '11px Arial'; ctx.fillText(upg.effect(lvl), lvlX, y + 44);

                    let cx = lx + lw - 120;
                    if (lvl >= upg.maxLevel) {
                        ctx.fillStyle = '#ffcc00'; ctx.font = 'bold 14px Arial'; ctx.textAlign = 'center';
                        ctx.fillText('✓ MAXED', cx + 50, y + ih / 2);
                    } else {
                        let cost = upg.costs[lvl]; let afford = saveData.rebirthTokens >= cost;
                        ctx.fillStyle = afford ? '#ff88ff' : '#ff4444'; ctx.font = 'bold 13px Arial'; ctx.textAlign = 'center';
                        ctx.fillText(`🌟 ${cost} RT`, cx + 50, y + ih / 2 - 5);
                        if (sel) { ctx.fillStyle = afford ? '#ffaaff' : '#ffaaaa'; ctx.font = '10px Arial'; ctx.fillText(afford ? '[ENTER]' : 'Need tokens', cx + 50, y + ih / 2 + 12); }
                    }
                    if (sel) { ctx.fillStyle = upg.color; ctx.font = 'bold 16px Arial'; ctx.textAlign = 'right'; ctx.fillText('▶', lx - 2, y + ih / 2 + 2); }
                }
            } else if (rebirthTab === 2) {
                // Enemy codex
                let allE = Object.keys(ENEMY_TYPES);
                let sy = 92, ih = 55, lx = 40, lw = CW - 80;

                ctx.fillStyle = '#aaaacc'; ctx.font = '14px Arial'; ctx.textAlign = 'center';
                ctx.fillText(`Enemies available at your Rebirth level (${saveData.rebirths}):`, CW / 2, sy - 5);

                allE.forEach((key, i) => {
                    let et = ENEMY_TYPES[key]; let y = sy + i * (ih + 3) + 8;
                    let unlocked = saveData.rebirths >= et.minRebirth;

                    ctx.fillStyle = unlocked ? 'rgba(20,10,35,0.8)' : 'rgba(10,10,20,0.5)';
                    roundRect(ctx, lx, y, lw, ih, 6, true, false);
                    ctx.strokeStyle = unlocked ? et.color : '#333344'; ctx.lineWidth = 1;
                    roundRect(ctx, lx, y, lw, ih, 6, false, true);

                    if (unlocked) {
                        // Mini enemy preview
                        ctx.fillStyle = et.color; ctx.beginPath();
                        let ex = lx + 30, ey = y + ih / 2;
                        for (let s = 0; s < 6; s++) {
                            let a = (s / 6) * Math.PI * 2 - Math.PI / 2;
                            ctx.lineTo(ex + Math.cos(a) * 12, ey + Math.sin(a) * 12);
                        }
                        ctx.closePath(); ctx.fill();
                        ctx.fillStyle = '#ff0000'; ctx.beginPath();
                        ctx.arc(ex, ey - 3, 3, 0, Math.PI * 2); ctx.fill();

                        ctx.fillStyle = et.color; ctx.font = 'bold 16px Arial'; ctx.textAlign = 'left';
                        ctx.fillText(et.name, lx + 55, y + 20);
                        ctx.fillStyle = '#aaaacc'; ctx.font = '12px Arial';
                        ctx.fillText(et.desc, lx + 55, y + 36);

                        ctx.fillStyle = '#888899'; ctx.font = '11px Arial'; ctx.textAlign = 'right';
                        ctx.fillText(`HP:${et.hp} SPD:${et.speed} PTS:${et.score}`, lx + lw - 10, y + 20);
                        ctx.fillText(`Shoots: ${et.shootRate > 0 ? 'Yes' : 'No'}`, lx + lw - 10, y + 36);
                    } else {
                        ctx.fillStyle = '#555566'; ctx.font = 'bold 16px Arial'; ctx.textAlign = 'left';
                        ctx.fillText(`🔒 ??? (Rebirth ${et.minRebirth} required)`, lx + 55, y + 20);
                        ctx.fillStyle = '#444455'; ctx.font = '12px Arial';
                        ctx.fillText('Unknown enemy type', lx + 55, y + 36);
                    }
                });
            }

            ctx.fillStyle = '#555577'; ctx.font = '12px Arial'; ctx.textAlign = 'center';
            ctx.fillText('←→/TAB Tabs • ↑↓ Select • ENTER Buy/Rebirth • ESC Back', CW / 2, CH - 5);
            drawNotification();
        }

        // ============ HANGAR ============
        function drawHangarScreen() {
            ctx.fillStyle = '#05051e'; ctx.fillRect(0, 0, CW, CH); drawStars(0.1);
            drawPanel(15, 5, CW - 30, 42, '#cc33ff');
            ctx.fillStyle = '#dd66ff'; ctx.font = 'bold 24px Arial'; ctx.textAlign = 'center';
            ctx.fillText('🎨 SHIP SKINS', CW / 2, 30);
            drawCreditsDisplay(CW - 30, 10);

            let skin = SHIP_SKINS[skinSelection];
            let unlocked = saveData.unlockedSkins[skinSelection];
            let equipped = saveData.selectedShipSkin === skinSelection;

            drawPanel(CW / 2 - 110, 60, 220, 200, skin.accentColor);
            if (unlocked) { drawShipAt(CW / 2, 175, 2.3, skin); }
            else { ctx.globalAlpha = 0.3; drawShipAt(CW / 2, 175, 2.3, skin); ctx.globalAlpha = 1;
                ctx.fillStyle = '#ffffff'; ctx.font = 'bold 36px Arial'; ctx.textAlign = 'center'; ctx.fillText('🔒', CW / 2, 175); }

            ctx.fillStyle = '#ffffff'; ctx.font = 'bold 24px Arial'; ctx.textAlign = 'center';
            ctx.fillText(skin.name, CW / 2, 290);

            if (equipped) { ctx.fillStyle = '#33ff66'; ctx.font = 'bold 16px Arial'; ctx.fillText('✓ EQUIPPED', CW / 2, 315); }
            else if (unlocked) { ctx.fillStyle = '#00ccff'; ctx.font = 'bold 16px Arial'; ctx.fillText('[ENTER to Equip]', CW / 2, 315); }
            else {
                ctx.fillStyle = credits >= skin.cost ? '#33ff66' : '#ff4444';
                ctx.font = 'bold 16px Arial'; ctx.fillText(`💰 ${skin.cost.toLocaleString()} CR`, CW / 2, 312);
                ctx.fillStyle = '#aaaacc'; ctx.font = '13px Arial'; ctx.fillText('[ENTER to Purchase]', CW / 2, 332);
            }

            ctx.fillStyle = '#ffffff'; ctx.font = 'bold 36px Arial'; ctx.textAlign = 'center';
            ctx.globalAlpha = 0.7; ctx.fillText('◀', CW / 2 - 170, 175); ctx.fillText('▶', CW / 2 + 170, 175); ctx.globalAlpha = 1;

            // Thumbnails
            let tw = 70, th = 80, total = SHIP_SKINS.length * (tw + 10) - 10;
            let startX = CW / 2 - total / 2;
            SHIP_SKINS.forEach((s, i) => {
                let tx = startX + i * (tw + 10), ty = 370;
                let sel = i === skinSelection, ul = saveData.unlockedSkins[i];
                ctx.fillStyle = sel ? 'rgba(30,30,80,0.95)' : 'rgba(10,10,35,0.7)';
                roundRect(ctx, tx, ty, tw, th, 5, true, false);
                ctx.strokeStyle = sel ? s.accentColor : '#333355'; ctx.lineWidth = sel ? 2 : 1;
                roundRect(ctx, tx, ty, tw, th, 5, false, true);
                if (ul) { drawShipAt(tx + tw / 2, ty + 32, 0.6, s); }
                else { ctx.globalAlpha = 0.3; drawShipAt(tx + tw / 2, ty + 32, 0.6, s); ctx.globalAlpha = 1;
                    ctx.fillStyle = '#fff'; ctx.font = '14px Arial'; ctx.textAlign = 'center'; ctx.fillText('🔒', tx + tw / 2, ty + 35); }
                ctx.fillStyle = '#aaaacc'; ctx.font = '9px Arial'; ctx.textAlign = 'center'; ctx.fillText(s.name, tx + tw / 2, ty + th - 6);
                if (saveData.selectedShipSkin === i) { ctx.fillStyle = '#33ff66'; ctx.font = 'bold 9px Arial'; ctx.fillText('✓', tx + tw / 2, ty + th + 8); }
            });

            ctx.fillStyle = '#555577'; ctx.font = '12px Arial'; ctx.textAlign = 'center';
            ctx.fillText('←→ Browse • ENTER Select • ESC Back', CW / 2, CH - 5);
            drawNotification();
        }

        // ============ STATS ============
        function drawStatsScreen() {
            ctx.fillStyle = '#05051e'; ctx.fillRect(0, 0, CW, CH); drawStars(0.1);
            drawPanel(15, 5, CW - 30, 42, '#33ccff');
            ctx.fillStyle = '#66ddff'; ctx.font = 'bold 24px Arial'; ctx.textAlign = 'center';
            ctx.fillText('📊 PILOT STATISTICS', CW / 2, 30);

            let stats = [
                { l: 'High Score', v: saveData.highScore.toLocaleString(), i: '🏆', c: '#ffcc00' },
                { l: 'Total Credits', v: credits.toLocaleString() + ' CR', i: '💰', c: '#ffaa00' },
                { l: 'Rebirth Level', v: saveData.rebirths.toString(), i: '🌟', c: '#ff88ff' },
                { l: 'Rebirth Tokens', v: saveData.rebirthTokens.toString(), i: '💎', c: '#dd66ff' },
                { l: 'Games Played', v: saveData.totalGamesPlayed.toString(), i: '🎮', c: '#33ccff' },
                { l: 'Asteroids Destroyed', v: saveData.totalAsteroidsDestroyed.toLocaleString(), i: '☄️', c: '#ff6633' },
                { l: 'Enemies Destroyed', v: saveData.totalEnemiesDestroyed.toLocaleString(), i: '👾', c: '#ff4488' },
                { l: 'Times Destroyed', v: saveData.totalDeaths.toString(), i: '💀', c: '#ff3333' },
                { l: 'Highest Wave', v: saveData.highestWave.toString(), i: '🌊', c: '#cc66ff' },
                { l: 'K/D Ratio', v: saveData.totalDeaths > 0 ? ((saveData.totalAsteroidsDestroyed + saveData.totalEnemiesDestroyed) / saveData.totalDeaths).toFixed(1) : 'N/A', i: '📈', c: '#33ff66' },
                { l: 'Skins Unlocked', v: `${saveData.unlockedSkins.filter(s => s).length}/${SHIP_SKINS.length}`, i: '🎨', c: '#dd66ff' },
                { l: 'Enemies Unlocked', v: `${getAvailableEnemies().length}/${Object.keys(ENEMY_TYPES).length}`, i: '👾', c: '#ff8844' }
            ];

            let sy = 60, ih = 46, lx = 100, lw = CW - 200;
            stats.forEach((s, i) => {
                let y = sy + i * (ih + 2);
                ctx.fillStyle = i % 2 === 0 ? 'rgba(15,15,45,0.6)' : 'rgba(10,10,35,0.4)';
                roundRect(ctx, lx, y, lw, ih, 4, true, false);
                ctx.fillStyle = s.c; ctx.font = '22px Arial'; ctx.textAlign = 'left'; ctx.fillText(s.i, lx + 12, y + ih / 2 + 3);
                ctx.fillStyle = '#bbbbdd'; ctx.font = '16px Arial'; ctx.fillText(s.l, lx + 45, y + ih / 2 + 2);
                ctx.fillStyle = s.c; ctx.font = 'bold 18px Arial'; ctx.textAlign = 'right';
                ctx.fillText(s.v, lx + lw - 15, y + ih / 2 + 2);
            });

            ctx.fillStyle = '#555577'; ctx.font = '12px Arial'; ctx.textAlign = 'center';
            ctx.fillText('ESC / ENTER to go back', CW / 2, CH - 5);
        }

        // ============ SETTINGS ============
        function drawSettingsScreen() {
            ctx.fillStyle = '#05051e'; ctx.fillRect(0, 0, CW, CH); drawStars(0.1);
            drawPanel(15, 5, CW - 30, 42, '#888888');
            ctx.fillStyle = '#aaaacc'; ctx.font = 'bold 24px Arial'; ctx.textAlign = 'center';
            ctx.fillText('⚙️ SETTINGS', CW / 2, 30);

            let sy = 80, ih = 55, lx = 150, lw = CW - 300;
            settingsItems.forEach((item, i) => {
                let y = sy + i * (ih + 8); let sel = i === settingsSelection;
                if (item === 'Back') {
                    let bc = sel ? '#666688' : 'rgba(20,20,60,0.8)';
                    ctx.fillStyle = bc; roundRect(ctx, lx, y, lw, ih, 8, true, false);
                    ctx.strokeStyle = sel ? '#ffffff' : '#666688'; ctx.lineWidth = sel ? 2 : 1;
                    roundRect(ctx, lx, y, lw, ih, 8, false, true);
                    ctx.fillStyle = sel ? '#ffffff' : '#aaaacc'; ctx.font = sel ? 'bold 18px Arial' : '18px Arial';
                    ctx.textAlign = 'center'; ctx.fillText('← Back', lx + lw / 2, y + ih / 2 + 1);
                    return;
                }
                ctx.fillStyle = sel ? 'rgba(30,30,80,0.95)' : 'rgba(10,10,35,0.7)';
                roundRect(ctx, lx, y, lw, ih, 8, true, false);
                ctx.strokeStyle = sel ? '#3366ff' : '#333355'; ctx.lineWidth = sel ? 2 : 1;
                roundRect(ctx, lx, y, lw, ih, 8, false, true);
                ctx.fillStyle = sel ? '#ffffff' : '#bbbbdd'; ctx.font = '16px Arial'; ctx.textAlign = 'left';
                ctx.fillText(item, lx + 20, y + ih / 2 + 1);
                ctx.textAlign = 'right'; let vx = lx + lw - 20;
                if (item === 'Master Volume') { drawProgressBar(vx - 180, y + ih / 2 - 6, 150, 12, saveData.settings.masterVolume, 1, '#33ff66');
                    ctx.fillStyle = '#ccccee'; ctx.font = '14px Arial'; ctx.fillText(`${Math.round(saveData.settings.masterVolume * 100)}%`, vx, y + ih / 2 + 1); }
                else if (item === 'SFX Volume') { drawProgressBar(vx - 180, y + ih / 2 - 6, 150, 12, saveData.settings.sfxVolume, 1, '#3399ff');
                    ctx.fillStyle = '#ccccee'; ctx.font = '14px Arial'; ctx.fillText(`${Math.round(saveData.settings.sfxVolume * 100)}%`, vx, y + ih / 2 + 1); }
                else if (item === 'Screen Shake') { ctx.fillStyle = saveData.settings.screenShake ? '#33ff66' : '#ff4444'; ctx.font = 'bold 16px Arial'; ctx.fillText(saveData.settings.screenShake ? 'ON' : 'OFF', vx, y + ih / 2 + 1); }
                else if (item === 'Show FPS') { ctx.fillStyle = saveData.settings.showFPS ? '#33ff66' : '#ff4444'; ctx.font = 'bold 16px Arial'; ctx.fillText(saveData.settings.showFPS ? 'ON' : 'OFF', vx, y + ih / 2 + 1); }
                else if (item === 'Particle Density') { let lb = { 0.5: 'Low', 1: 'Normal', 1.5: 'High', 2: 'Ultra' }; ctx.fillStyle = '#ccccee'; ctx.font = 'bold 16px Arial'; ctx.fillText(lb[saveData.settings.particleDensity] || 'Normal', vx, y + ih / 2 + 1); }
            });
            ctx.fillStyle = '#555577'; ctx.font = '12px Arial'; ctx.textAlign = 'center';
            ctx.fillText('↑↓ Select • ←→ Adjust • ESC Back', CW / 2, CH - 5);
        }

        // ============ HUD ============
        function drawHUD() {
            // Score
            ctx.fillStyle = 'rgba(0,0,40,0.7)'; roundRect(ctx, CW / 2 - 120, 6, 240, 38, 8, true, false);
            ctx.strokeStyle = '#3366ff'; ctx.lineWidth = 2; roundRect(ctx, CW / 2 - 120, 6, 240, 38, 8, false, true);
            ctx.fillStyle = '#ffffff'; ctx.font = 'bold 22px Arial'; ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
            ctx.fillText(`SCORE: ${score.toLocaleString()}`, CW / 2, 25);

            // Wave
            ctx.fillStyle = 'rgba(0,0,40,0.7)'; roundRect(ctx, 8, 8, 110, 34, 8, true, false);
            ctx.strokeStyle = '#aa33ff'; ctx.lineWidth = 2; roundRect(ctx, 8, 8, 110, 34, 8, false, true);
            ctx.fillStyle = '#cc66ff'; ctx.font = 'bold 20px Arial'; ctx.textAlign = 'left'; ctx.fillText(`WAVE ${wave}`, 20, 28);

            // Power
            ctx.fillStyle = 'rgba(0,0,40,0.7)'; roundRect(ctx, CW - 160, 8, 150, 34, 8, true, false);
            ctx.strokeStyle = '#ff1493'; ctx.lineWidth = 2; roundRect(ctx, CW - 160, 8, 150, 34, 8, false, true);
            ctx.fillStyle = '#ff1493'; ctx.font = 'bold 14px Arial'; ctx.textAlign = 'left';
            let pl = ship.powerLevel + getUpg('multiShot');
            let fc = getFireCount(pl);
            ctx.fillText(`POWER: ${fc} shots`, CW - 148, 28);

            // Credits preview
            ctx.fillStyle = 'rgba(0,0,40,0.7)'; roundRect(ctx, CW - 160, 48, 150, 22, 6, true, false);
            ctx.fillStyle = '#ffcc00'; ctx.font = '12px Arial'; ctx.textAlign = 'left';
            ctx.fillText(`💰 +${Math.floor(score / 10).toLocaleString()} CR`, CW - 148, 62);

            // Health
            let bx = 18, by = CH - 32, bw = 200, bh = 16;
            ctx.fillStyle = 'rgba(0,0,40,0.7)'; roundRect(ctx, bx - 4, by - 40, bw + 8, bh + 48, 5, true, false);
            ctx.fillStyle = '#99ccff'; ctx.font = '11px Arial'; ctx.textAlign = 'left';
            ctx.fillText(`HULL: ${ship.health}/${ship.maxHealth}`, bx, by - 6);
            ctx.fillStyle = '#333333'; roundRect(ctx, bx, by, bw, bh, 3, true, false);
            let hp = ship.health / ship.maxHealth;
            let hc = hp > 0.5 ? '#33ff66' : hp > 0.25 ? '#ffff00' : '#ff3333';
            if (hp > 0) { ctx.fillStyle = hc; roundRect(ctx, bx, by, bw * hp, bh, 3, true, false); }
            ctx.strokeStyle = '#fff'; ctx.lineWidth = 2; roundRect(ctx, bx, by, bw, bh, 3, false, true);

            // Lives
            ctx.fillStyle = '#ffffff'; ctx.font = 'bold 12px Arial'; ctx.fillText('LIVES:', bx, by - 28);
            for (let i = 0; i < ship.lives; i++) {
                let cx = bx + 55 + i * 22, cy = by - 30;
                ctx.fillStyle = '#00ffff'; ctx.beginPath(); ctx.moveTo(cx, cy - 7); ctx.lineTo(cx - 5, cy + 4); ctx.lineTo(cx + 5, cy + 4); ctx.closePath(); ctx.fill();
            }

            // Combo
            if (combo > 1 && comboTimer > 0) {
                ctx.fillStyle = '#ffff00'; ctx.font = 'bold 26px Arial'; ctx.textAlign = 'center';
                ctx.shadowBlur = 10; ctx.shadowColor = '#ffff00';
                ctx.fillText(`COMBO x${combo}!`, CW / 2, 72); ctx.shadowBlur = 0;
            }

            // Shield
            if (ship.shieldActive) {
                ctx.fillStyle = '#00ffff'; ctx.font = 'bold 14px Arial'; ctx.textAlign = 'center';
                ctx.fillText(`SHIELD: ${Math.ceil(ship.shieldTimer / 60)}s`, CW / 2, 95);
            }

            // Wave progress
            ctx.fillStyle = '#777799'; ctx.font = '10px Arial'; ctx.textAlign = 'center';
            ctx.fillText(`Wave: ${asteroidsDestroyed}/${asteroidsToDestroy}`, CW / 2, CH - 12);
            drawProgressBar(CW / 2 - 70, CH - 8, 140, 6, asteroidsDestroyed, asteroidsToDestroy, '#aa33ff');

            // Rebirth indicator
            if (saveData.rebirths > 0) {
                ctx.fillStyle = '#ff88ff'; ctx.font = '11px Arial'; ctx.textAlign = 'right';
                ctx.fillText(`⭐ R${saveData.rebirths}`, CW - 10, CH - 5);
            }

            ctx.fillStyle = '#444466'; ctx.font = '11px Arial'; ctx.textAlign = 'right';
            ctx.fillText('ESC Pause', CW - 10, CH - 18);
        }

        function drawWaveAnnouncement() {
            if (waveAnnounceTimer <= 0) return;
            let a = Math.min(1, waveAnnounceTimer / 20);
            if (waveAnnounceTimer > 40) { ctx.fillStyle = `rgba(0,0,0,${Math.min(0.4, waveAnnounceTimer / 300)})`; ctx.fillRect(0, 0, CW, CH); }
            ctx.globalAlpha = a; ctx.fillStyle = '#ffffff'; ctx.font = 'bold 56px Arial'; ctx.textAlign = 'center';
            ctx.shadowBlur = 20; ctx.shadowColor = '#3366ff';
            ctx.fillText(`WAVE ${wave}`, CW / 2, CH / 2 - 20);
            ctx.font = 'bold 20px Arial'; ctx.fillStyle = '#ffff00'; ctx.shadowColor = '#ffaa00';
            let enemies = getAvailableEnemies();
            let sub = enemies.length > 1 ? 'Enemies approaching!' : 'Incoming asteroids!';
            ctx.fillText(sub, CW / 2, CH / 2 + 20);
            ctx.shadowBlur = 0; ctx.globalAlpha = 1;
        }

        // ============ PAUSE ============
        function drawPauseScreen() {
            drawGameFrame(false);
            ctx.fillStyle = 'rgba(0,0,10,0.75)'; ctx.fillRect(0, 0, CW, CH);
            let pw = 320, ph = 260, px = CW / 2 - pw / 2, py = CH / 2 - ph / 2;
            drawPanel(px, py, pw, ph, '#3366ff');
            ctx.fillStyle = '#ffffff'; ctx.font = 'bold 32px Arial'; ctx.textAlign = 'center';
            ctx.fillText('⏸️ PAUSED', CW / 2, py + 45);
            ctx.fillStyle = '#aaaacc'; ctx.font = '14px Arial';
            ctx.fillText(`Score: ${score.toLocaleString()} • Wave ${wave}`, CW / 2, py + 75);
            let msy = py + 100, bw = 220, bh = 38;
            pauseMenuItems.forEach((item, i) => {
                let y = msy + i * (bh + 10); let sel = i === pauseMenuSelection;
                let c = i === 2 ? '#ff4444' : '#3366ff';
                if (sel) { ctx.shadowBlur = 10; ctx.shadowColor = c; }
                ctx.fillStyle = sel ? c : 'rgba(20,20,60,0.8)';
                roundRect(ctx, CW / 2 - bw / 2, y, bw, bh, 8, true, false);
                ctx.strokeStyle = sel ? '#ffffff' : c; ctx.lineWidth = sel ? 2 : 1;
                roundRect(ctx, CW / 2 - bw / 2, y, bw, bh, 8, false, true); ctx.shadowBlur = 0;
                ctx.fillStyle = sel ? '#ffffff' : '#aaaacc'; ctx.font = sel ? 'bold 18px Arial' : '16px Arial';
                ctx.textAlign = 'center'; ctx.fillText(item, CW / 2, y + bh / 2 + 1);
            });
        }

        // ============ GAME OVER ============
        function drawGameOverScreen() {
            ctx.fillStyle = '#05051e'; ctx.fillRect(0, 0, CW, CH); drawStars(0.2);
            ctx.fillStyle = 'rgba(0,0,0,0.5)'; ctx.fillRect(0, 0, CW, CH);
            particles.forEach(p => { p.update(); p.draw(); });
            particles = particles.filter(p => p.lifetime > 0);

            let pw = 420, ph = 380, px = CW / 2 - pw / 2, py = CH / 2 - ph / 2;
            drawPanel(px, py, pw, ph, '#ff3333');

            ctx.shadowBlur = 20; ctx.shadowColor = '#ff0000';
            ctx.fillStyle = '#ff3333'; ctx.font = 'bold 48px Arial'; ctx.textAlign = 'center';
            ctx.fillText('GAME OVER', CW / 2, py + 50); ctx.shadowBlur = 0;

            let stats = [
                { l: 'Final Score', v: score.toLocaleString(), c: '#ffffff' },
                { l: 'Wave', v: wave.toString(), c: '#cc66ff' },
                { l: 'Asteroids', v: sessionAsteroidsDestroyed.toString(), c: '#ff6633' },
                { l: 'Enemies', v: sessionEnemiesDestroyed.toString(), c: '#ff4488' },
                { l: 'Credits Earned', v: `+${creditsEarned.toLocaleString()} CR`, c: '#ffcc00' }
            ];
            let ssy = py + 80;
            stats.forEach((s, i) => {
                ctx.fillStyle = '#8888aa'; ctx.font = '14px Arial'; ctx.textAlign = 'left';
                ctx.fillText(s.l + ':', px + 50, ssy + i * 30);
                ctx.fillStyle = s.c; ctx.font = 'bold 18px Arial'; ctx.textAlign = 'right';
                ctx.fillText(s.v, px + pw - 50, ssy + i * 30);
            });

            if (isNewHigh) {
                let t = Date.now() * 0.001;
                ctx.fillStyle = `rgb(${Math.floor(200 + 55 * Math.sin(t * 5))}, ${Math.floor(130 + 55 * Math.sin(t * 5))}, 0)`;
                ctx.font = 'bold 24px Arial'; ctx.textAlign = 'center';
                ctx.fillText('★ NEW HIGH SCORE! ★', CW / 2, py + 250);
            }

            let t = Date.now() * 0.001;
            ctx.fillStyle = `rgb(${Math.floor(128 + 127 * Math.sin(t * 3))}, ${Math.floor(128 + 127 * Math.sin(t * 3))}, 255)`;
            ctx.font = 'bold 20px Arial'; ctx.textAlign = 'center';
            ctx.fillText('[ ENTER to Retry ]', CW / 2, py + 310);
            ctx.fillStyle = '#888888'; ctx.font = '15px Arial';
            ctx.fillText('ESC for Menu', CW / 2, py + 340);
            ctx.fillStyle = '#ffcc00'; ctx.font = 'bold 14px Arial';
            ctx.fillText(`Total Credits: ${credits.toLocaleString()} CR`, CW / 2, py + ph - 10);
        }

        // ============ GAME RESET ============
        function resetGame() {
            resetShip(); lasers = []; asteroids = []; particles = [];
            powerUps = []; scorePopups = []; enemies = []; enemyLasers = []; missiles = [];
            score = 0; wave = 1; combo = 0; comboTimer = 0;
            waveAnnounceTimer = 120; asteroidsDestroyed = 0; asteroidsToDestroy = 10;
            gameTime = 0; spawnTimer = 0; enemySpawnTimer = 0; missileTimer = 0;
            isNewHigh = false; sessionAsteroidsDestroyed = 0; sessionEnemiesDestroyed = 0;
            creditsEarned = 0;
        }

        // ============ GAME FRAME ============
        function drawGameFrame(update = true) {
            if (update) {
                gameTime++; updateShip();
                if (keys[' '] || keys['Space']) shipShoot();
                if (waveAnnounceTimer > 0) waveAnnounceTimer--;

                // Spawn asteroids
                let spawnRate = Math.max(15, 60 - wave * 4);
                spawnTimer++;
                if (spawnTimer >= spawnRate) { spawnTimer = 0; for (let i = 0; i < Math.min(wave, 4); i++) asteroids.push(new Asteroid()); }

                // Spawn enemies
                let available = getAvailableEnemies();
                if (available.length > 0 && wave >= 2) {
                    enemySpawnTimer++;
                    let eRate = Math.max(60, 200 - wave * 10);
                    if (enemySpawnTimer >= eRate) {
                        enemySpawnTimer = 0;
                        let type = available[Math.floor(Math.random() * available.length)];
                        enemies.push(new Enemy(type));
                        if (ENEMY_TYPES[type].hp >= 15) playSound('boss');
                    }
                }

                // Wave completion
                if (asteroidsDestroyed >= asteroidsToDestroy) {
                    wave++; asteroidsDestroyed = 0; asteroidsToDestroy = 10 + wave * 5;
                    waveAnnounceTimer = 120;
                    let scoreMult = 1 + getUpg('scoreMultiplier') * 0.1;
                    let bonus = Math.floor(wave * 500 * scoreMult);
                    score += bonus;
                    addScorePopup(CW / 2, CH / 2, `WAVE BONUS +${bonus}`, '#ff66ff');
                }

                if (comboTimer > 0) comboTimer--; else combo = 0;

                lasers.forEach(l => l.update()); lasers = lasers.filter(l => l.alive);
                asteroids.forEach(a => a.update()); asteroids = asteroids.filter(a => a.alive);
                enemies.forEach(e => e.update()); enemies = enemies.filter(e => e.alive);
                missiles.forEach(m => m.update()); missiles = missiles.filter(m => m.alive);
                particles.forEach(p => p.update()); particles = particles.filter(p => p.lifetime > 0);
                powerUps.forEach(pu => pu.update()); powerUps = powerUps.filter(pu => pu.alive);
                scorePopups.forEach(sp => { sp.lifetime--; sp.y -= 1.5; });
                scorePopups = scorePopups.filter(sp => sp.lifetime > 0);

                // Enemy lasers update
                enemyLasers.forEach(el => { el.x += el.vx; el.y += el.vy; if (el.x < -10 || el.x > CW + 10 || el.y < -10 || el.y > CH + 10) el.alive = false; });
                enemyLasers = enemyLasers.filter(el => el.alive);

                let scoreMult = 1 + getUpg('scoreMultiplier') * 0.1;
                let dropRate = 0.15 + getUpg('luckyDrops') * 0.03;
                let dmgReduction = 1 - getRUpg('cosmicArmor') * 0.05;
                let voidDmg = getRUpg('voidEnergy') * 0.15;

                // Laser vs Asteroid
                for (let i = lasers.length - 1; i >= 0; i--) {
                    let laser = lasers[i]; if (!laser || !laser.alive) continue;
                    for (let j = asteroids.length - 1; j >= 0; j--) {
                        let ast = asteroids[j]; if (!ast || !ast.alive) continue;
                        if (laser.hitTargets.has(ast.id)) continue;
                        if (circleCollision(laser.x, laser.y, 5, ast.x, ast.y, ast.radius)) {
                            laser.hitTargets.add(ast.id);
                            let destroyed = ast.hit(laser.damage);
                            if (destroyed) {
                                createExplosion(ast.x, ast.y, ast.color, 15 + ast.radius, 4);
                                startShake(ast.radius * 0.2, 8); playSound('explosion');
                                combo++; comboTimer = 90;
                                let earned = Math.floor(ast.scoreValue * Math.min(combo, 10) * scoreMult);
                                score += earned; addScorePopup(ast.x, ast.y, `+${earned}`, laser.isCrit ? '#ff3333' : '#ffff00');
                                asteroidsDestroyed++; sessionAsteroidsDestroyed++;
                                // Void energy
                                if (voidDmg > 0) {
                                    [...asteroids, ...enemies].forEach(target => {
                                        if (target !== ast && target.alive !== false) {
                                            let dx = target.x - ast.x, dy = target.y - ast.y;
                                            if (Math.sqrt(dx * dx + dy * dy) < 80) target.hit(Math.ceil(laser.damage * voidDmg));
                                        }
                                    });
                                }
                                asteroids.push(...ast.split());
                                if (Math.random() < dropRate) powerUps.push(new PowerUp(ast.x, ast.y));
                                asteroids.splice(j, 1);
                            } else { createExplosion(laser.x, laser.y, '#ffff00', 5, 2); playSound('hit'); }
                            if (laser.pierceLeft <= 0) { laser.alive = false; lasers.splice(i, 1); break; }
                            else laser.pierceLeft--;
                        }
                    }
                }

                // Laser/Missile vs Enemy
                let allProjectiles = [...lasers, ...missiles];
                for (let i = allProjectiles.length - 1; i >= 0; i--) {
                    let proj = allProjectiles[i]; if (!proj || !proj.alive) continue;
                    for (let j = enemies.length - 1; j >= 0; j--) {
                        let en = enemies[j]; if (!en || !en.alive) continue;
                        if (proj.hitTargets && proj.hitTargets.has(en.id)) continue;
                        if (circleCollision(proj.x, proj.y, 8, en.x, en.y, en.radius)) {
                            if (proj.hitTargets) proj.hitTargets.add(en.id);
                            let destroyed = en.hit(proj.damage);
                            if (destroyed) {
                                createExplosion(en.x, en.y, en.color, 25, 6);
                                startShake(5, 10); playSound('explosion');
                                combo++; comboTimer = 90;
                                let earned = Math.floor(en.score * Math.min(combo, 10) * scoreMult);
                                score += earned; addScorePopup(en.x, en.y, `+${earned}`, '#ff88ff');
                                sessionEnemiesDestroyed++; asteroidsDestroyed++;
                                if (Math.random() < dropRate * 1.5) powerUps.push(new PowerUp(en.x, en.y));
                                enemies.splice(j, 1);
                            } else { createExplosion(proj.x, proj.y, en.color, 5, 2); playSound('hit'); }
                            if (proj instanceof Missile) { proj.alive = false; break; }
                            if (proj.pierceLeft !== undefined && proj.pierceLeft <= 0) { proj.alive = false; break; }
                            else if (proj.pierceLeft !== undefined) proj.pierceLeft--;
                        }
                    }
                }
                // Clean up
                lasers = lasers.filter(l => l.alive);
                missiles = missiles.filter(m => m.alive);

                // Asteroid/Enemy vs Ship
                if (ship.invincible <= 0) {
                    let allDanger = [...asteroids.map((a, idx) => ({ obj: a, arr: asteroids, idx, radius: a.radius })),
                                     ...enemies.map((e, idx) => ({ obj: e, arr: enemies, idx, radius: e.radius }))];
                    for (let d of allDanger) {
                        if (!d.obj.alive) continue;
                        if (circleCollision(ship.x, ship.y, ship.width / 2 - 5, d.obj.x, d.obj.y, d.radius)) {
                            if (!ship.shieldActive) {
                                let dmg = Math.floor(35 * dmgReduction);
                                ship.health -= dmg; ship.invincible = 60;
                                startShake(10, 15); createExplosion(ship.x, ship.y, '#ff3333', 25, 6); playSound('explosion');
                                if (ship.health <= 0) {
                                    ship.lives--; ship.health = ship.maxHealth;
                                    ship.powerLevel = Math.max(1, ship.powerLevel - 1);
                                    if (ship.lives <= 0) {
                                        createExplosion(ship.x, ship.y, '#ff6600', 50, 8); playSound('gameover');
                                        endGame(true); state = 'gameover';
                                    }
                                }
                            }
                            d.obj.alive = false; break;
                        }
                    }
                    asteroids = asteroids.filter(a => a.alive);
                    enemies = enemies.filter(e => e.alive);
                }

                // Enemy laser vs Ship
                if (ship.invincible <= 0) {
                    for (let i = enemyLasers.length - 1; i >= 0; i--) {
                        let el = enemyLasers[i]; if (!el.alive) continue;
                        if (circleCollision(ship.x, ship.y, ship.width / 2 - 3, el.x, el.y, 5)) {
                            el.alive = false;
                            if (!ship.shieldActive) {
                                let dmg = Math.floor(20 * dmgReduction);
                                ship.health -= dmg; ship.invincible = 30;
                                startShake(5, 8); createExplosion(ship.x, ship.y, '#ff3333', 10, 3); playSound('hit');
                                if (ship.health <= 0) {
                                    ship.lives--; ship.health = ship.maxHealth;
                                    if (ship.lives <= 0) { playSound('gameover'); endGame(true); state = 'gameover'; }
                                }
                            }
                        }
                    }
                    enemyLasers = enemyLasers.filter(el => el.alive);
                }

                // Power-up vs Ship
                for (let i = powerUps.length - 1; i >= 0; i--) {
                    let pu = powerUps[i]; if (!pu.alive) continue;
                    if (circleCollision(ship.x, ship.y, ship.width / 2 + 5, pu.x, pu.y, pu.radius)) {
                        if (pu.type === 'power') { ship.powerLevel = Math.min(12, ship.powerLevel + 1); ship.shootDelay = Math.max(2, 12 - getUpg('fireRate')); addScorePopup(pu.x, pu.y - 20, 'POWER UP!', '#ff1493'); }
                        else if (pu.type === 'health') { ship.health = Math.min(ship.maxHealth, ship.health + 30); addScorePopup(pu.x, pu.y - 20, 'REPAIR!', '#33ff66'); }
                        else if (pu.type === 'shield') { ship.shieldTimer = Math.floor(300 * (1 + getUpg('shieldDuration') * 0.2)); addScorePopup(pu.x, pu.y - 20, 'SHIELD!', '#00ffff'); }
                        else if (pu.type === 'bomb') {
                            // Bomb - destroy nearby
                            [...asteroids, ...enemies].forEach(t => {
                                let dx = t.x - pu.x, dy = t.y - pu.y;
                                if (Math.sqrt(dx * dx + dy * dy) < 150) { t.alive = false; createExplosion(t.x, t.y, '#ff8800', 15, 4); score += 50; }
                            });
                            createExplosion(pu.x, pu.y, '#ff8800', 40, 8);
                            addScorePopup(pu.x, pu.y - 20, 'BOOM!', '#ff8800');
                            startShake(8, 12);
                        }
                        else if (pu.type === 'credit') {
                            let bonus = 100 + wave * 20;
                            credits += bonus; saveData.credits = credits; saveSave();
                            addScorePopup(pu.x, pu.y - 20, `+${bonus} CR!`, '#ffcc00');
                        }
                        createExplosion(pu.x, pu.y, pu.colorMap[pu.type], 15, 3);
                        playSound('powerup'); powerUps.splice(i, 1);
                    }
                }
            }

            // ====== DRAW ======
            updateShake(); ctx.save(); ctx.translate(screenShakeX, screenShakeY);
            ctx.fillStyle = '#05051e'; ctx.fillRect(-10, -10, CW + 20, CH + 20);
            drawStars();
            powerUps.forEach(pu => pu.draw());
            asteroids.forEach(a => a.draw());
            enemies.forEach(e => e.draw());
            lasers.forEach(l => l.draw());
            missiles.forEach(m => m.draw());

            // Enemy lasers
            enemyLasers.forEach(el => {
                ctx.shadowBlur = 10; ctx.shadowColor = el.color;
                ctx.strokeStyle = el.color; ctx.lineWidth = 3;
                ctx.beginPath(); ctx.moveTo(el.x, el.y); ctx.lineTo(el.x - el.vx, el.y - el.vy); ctx.stroke();
                ctx.strokeStyle = '#ffffff'; ctx.lineWidth = 1;
                ctx.beginPath(); ctx.moveTo(el.x, el.y); ctx.lineTo(el.x - el.vx, el.y - el.vy); ctx.stroke();
                ctx.shadowBlur = 0;
            });

            particles.forEach(p => p.draw());
            drawShipAt();

            scorePopups.forEach(sp => {
                ctx.globalAlpha = sp.lifetime / sp.maxLifetime;
                ctx.fillStyle = sp.color; ctx.font = 'bold 18px Arial'; ctx.textAlign = 'center';
                ctx.shadowBlur = 5; ctx.shadowColor = sp.color;
                ctx.fillText(sp.text, sp.x, sp.y); ctx.shadowBlur = 0; ctx.globalAlpha = 1;
            });

            // Enemy warning (radar upgrade)
            if (getRUpg('enemyRadar') >= 3) {
                enemies.forEach(e => {
                    if (e.y < 0) {
                        ctx.fillStyle = 'rgba(255,0,0,0.6)'; ctx.font = 'bold 12px Arial'; ctx.textAlign = 'center';
                        ctx.fillText('⚠️', e.x, 15);
                    }
                });
            }

            drawHUD(); drawWaveAnnouncement(); ctx.restore();
        }

        // ============ MAIN LOOP ============
        let fpsFrames = 0, fpsTime = 0, fpsDisplay = 60, lastFrame = performance.now();

        function gameLoop(ts) {
            let dt = ts - lastFrame; lastFrame = ts;
            fpsFrames++; fpsTime += dt;
            if (fpsTime >= 1000) { fpsDisplay = fpsFrames; fpsFrames = 0; fpsTime = 0; }

            ctx.clearRect(0, 0, CW, CH);

            if (state === 'menu') drawMainMenu();
            else if (state === 'upgrades') drawUpgradeScreen();
            else if (state === 'rebirth') drawRebirthScreen();
            else if (state === 'hangar') drawHangarScreen();
            else if (state === 'stats') drawStatsScreen();
            else if (state === 'settings') drawSettingsScreen();
            else if (state === 'playing') drawGameFrame(true);
            else if (state === 'paused') drawPauseScreen();
            else if (state === 'gameover') drawGameOverScreen();

            if (saveData.settings.showFPS) {
                ctx.fillStyle = '#33ff33'; ctx.font = '12px monospace'; ctx.textAlign = 'left';
                ctx.fillText(`FPS: ${fpsDisplay}`, 5, CH - 3);
            }

            requestAnimationFrame(gameLoop);
        }

        requestAnimationFrame(gameLoop);
    </script>
</body>
</html>
