<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>File Crusher 🤯</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>
    <style>
        /* ------------------ CSS: Standard Blue Theme 💙 & Organization ------------------ */
        :root {
            --primary-blue: #007bff; /* Standard, strong blue */
            --light-grey: #f8f9fa;   /* Very light background */
            --dark-text: #212529;
            --card-bg: #ffffff;
            --border-color: #ced4da;
            --success-green: #28a745; /* For progress bar */
        }

        body {
            font-family: 'Arial', sans-serif;
            background-color: var(--light-grey);
            color: var(--dark-text);
            display: flex;
            justify-content: center;
            align-items: flex-start;
            min-height: 100vh;
            margin: 0;
            padding: 40px 15px;
        }

        .container {
            background-color: var(--card-bg);
            padding: 35px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
            width: 95%;
            max-width: 900px;
        }
        
        h1 { color: var(--primary-blue); text-align: center; margin-bottom: 10px; font-size: 2.2em; }
        .subtitle { text-align: center; color: #6c757d; margin-bottom: 30px; }

        /* Drop Area */
        #dropArea {
            border: 3px dashed var(--primary-blue);
            border-radius: 10px;
            padding: 50px 20px;
            text-align: center;
            cursor: pointer;
            margin-bottom: 30px;
            transition: background-color 0.3s;
        }
        #dropArea:hover { background-color: #e9ecef; border-color: #0056b3; }
        #fileInput { display: none; }
        .icon { font-size: 3.5em; color: var(--primary-blue); margin-bottom: 15px; }

        /* Progress Bar Container */
        #progressContainer {
            margin-bottom: 25px;
            display: none; 
            align-items: center;
        }
        .progress-bar {
            flex-grow: 1;
            height: 25px;
            background-color: var(--border-color);
            border-radius: 5px;
            overflow: hidden;
            margin-right: 15px;
        }
        .progress-bar-fill {
            height: 100%;
            width: 0%;
            background-color: var(--success-green);
            transition: width 0.1s linear; 
        }
        #progressText {
            width: 80px;
            font-weight: bold;
            color: var(--dark-text);
        }
        
        .cancel-button {
            padding: 7px 14px; background-color: #dc3545; color: white; border: none;
            border-radius: 5px; cursor: pointer; font-size: 0.9em; transition: background-color 0.3s;
        }
        .cancel-button:hover { background-color: #c82333; }

        /* Info Card and Tabs (Styles remain similar to previous version) */
        .info-card { background-color: #e6f3ff; border-left: 5px solid var(--primary-blue); padding: 15px; border-radius: 8px; margin-bottom: 25px; font-size: 0.95em; }
        .info-card span { font-weight: 700; color: var(--dark-text); }
        .tabs { display: flex; border-bottom: 2px solid var(--border-color); margin-bottom: 20px; }
        .tab-button { padding: 12px 18px; cursor: pointer; border: none; background-color: transparent; color: var(--dark-text); font-weight: 600; transition: color 0.3s, border-bottom 0.3s; border-bottom: 3px solid transparent; margin-bottom: -2px; font-size: 1em; }
        .tab-button.active { color: var(--primary-blue); border-bottom: 3px solid var(--primary-blue); }
        .content-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px; border-bottom: 1px solid #dee2e6; padding-bottom: 8px; }
        .action-button { padding: 7px 14px; background-color: var(--primary-blue); color: white; border: none; border-radius: 5px; cursor: pointer; font-size: 0.9em; transition: background-color 0.3s; }
        .action-button:hover { background-color: #0056b3; }
        .content-area { border: 1px solid var(--border-color); border-radius: 8px; padding: 15px; background-color: #fcfcfc; min-height: 250px; overflow: auto; white-space: pre-wrap; }
        pre { white-space: pre-wrap; word-wrap: break-word; margin: 0; font-size: 0.85em; }
        
        /* ZIP List */
        #zipContent li { padding: 10px; border-bottom: 1px solid #eee; cursor: pointer; transition: background-color 0.2s; display: flex; justify-content: space-between; align-items: center; }
        #zipContent li:hover { background-color: #f1f8ff; }
        .file-name { font-weight: 600; }
        .file-size { font-size: 0.8em; color: #6c757d; }
        
        /* Security Ban Screen */
        #banScreen { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background-color: #000000; color: #dc3545; display: flex; flex-direction: column; justify-content: center; align-items: center; text-align: center; font-size: 3em; z-index: 9999; font-weight: bold; }
        #banScreen p { font-size: 0.5em; color: #aaa; margin-top: 10px; }
    </style>
</head>
<body onload="checkBanStatus()">
    
    <div id="banScreen" style="display: none;">
        <span style="font-size: 5em;">🚫</span>
        <p>SECURITY VIOLATION!</p>
        <p>Access to this utility has been permanently blocked due to policy breach.</p>
    </div>
    
    <div class="container">
        <h1>File Crusher 🤯</h1>
        <p class="subtitle">Crush the kernel, extract the code, and ensure compliance.</p>
        
        <div id="dropArea">
            <div class="icon">📂</div>
            <p>Click or Drag a File to Analyze</p>
            <input type="file" id="fileInput" multiple>
        </div>

        <div id="outputArea" style="display: none;">
            
            <div id="progressContainer">
                <div class="progress-bar">
                    <div id="progressBarFill" class="progress-bar-fill"></div>
                </div>
                <span id="progressText">0%</span>
                <button class="cancel-button" onclick="cancelOrder()">Cancel Order ❌</button>
            </div>
            
            <div class="info-card">
                File: <span id="fileName">...</span> | Size: <span id="fileSize">...</span> | Type: <span id="fileType">...</span>
            </div>
            
            <div class="tabs">
                <button class="tab-button active" onclick="showTab('normal-view')">Normal View 🖼️</button>
                <button class="tab-button" onclick="showTab('text-view')">Extracted Code/Text ✍️</button>
                <button class="tab-button" onclick="showTab('raw-view')">File Kernel (HEX) 💻</button>
                <button class="tab-button" onclick="showTab('zip-view')" id="zipTab" style="display: none;">Archive Contents 🗜️</button>
            </div>

            <div id="normal-view" class="tab-content" style="display: block;">
                <div class="content-header"> 
                    <h4>Direct View</h4> 
                    <button class="action-button" onclick="saveNormalContent()">Save Original File 💾</button> 
                </div>
                <div class="content-area"> <div id="normalContent"></div> </div>
            </div>
            <div id="text-view" class="tab-content" style="display: none;">
                <div class="content-header"> 
                    <h4>Extracted Text</h4> 
                    <button class="action-button" onclick="copyContent('textContent')">Copy Content 📋</button> 
                </div>
                <div class="content-area"> <pre id="textContent">...</pre> </div>
            </div>
            <div id="raw-view" class="tab-content" style="display: none;">
                 <div class="content-header"> 
                    <h4>Raw Data (HEX View)</h4> 
                    <button class="action-button" onclick="copyContent('rawContent')">Copy Content 📋</button> 
                </div>
                <div class="content-area"> <pre id="rawContent">...</pre> </div>
            </div>
            <div id="zip-view" class="tab-content" style="display: none;">
                <div class="content-header"> 
                    <h4>Files in Archive (Click to view content)</h4> 
                    <button class="action-button" onclick="saveAllZipContent()">Save All to ZIP 💾</button> 
                </div>
                <div class="content-area"> <ul id="zipContent"></ul> </div>
            </div>

            <p id="messageArea" style="text-align: center; margin-top: 20px; color: #dc3545;"></p>
        </div>
    </div>

    <script>
        // --- Security & Compliance Variables ---
        // Token Policy: 4,000,000 tokens ≈ 16 MB limit for code archives
        const MAX_APK_SIZE_BYTES = 16000000; 
        const MAX_IMAGE_SIZE_BYTES = 5 * 1024 * 1024; 
        const MAX_VIOLATIONS = 3; 
        
        let violationCount = parseInt(localStorage.getItem('fc_violations') || '0', 10);
        const banStatus = localStorage.getItem('fc_banned');
        let progressInterval = null; 
        let isCancelled = false; 

        // --- UI Elements ---
        const progressBarFill = document.getElementById('progressBarFill');
        const progressText = document.getElementById('progressText');
        const progressContainer = document.getElementById('progressContainer');
        const dropArea = document.getElementById('dropArea');
        const fileInput = document.getElementById('fileInput');
        const outputArea = document.getElementById('outputArea');
        const zipTab = document.getElementById('zipTab');
        const zipContentList = document.getElementById('zipContent');
        
        let currentFile = null;
        let currentZip = null; 

        // --- Security Functions (Unchanged) ---
        function checkBanStatus() {
            if (banStatus === 'true' || violationCount >= MAX_VIOLATIONS) {
                document.getElementById('banScreen').style.display = 'flex';
                document.body.style.overflow = 'hidden'; 
                return true;
            }
            return false;
        }

        function recordViolation(message) {
            violationCount++;
            localStorage.setItem('fc_violations', violationCount);
            
            const remaining = MAX_VIOLATIONS - violationCount;
            stopProgress(); 
            
            if (violationCount >= MAX_VIOLATIONS) {
                localStorage.setItem('fc_banned', 'true');
                document.getElementById('messageArea').textContent = "⛔️ Maximum attempts reached. The application is now permanently blocked.";
                setTimeout(() => { checkBanStatus(); }, 1000);
            } else {
                document.getElementById('messageArea').textContent = `⛔️ Security Policy Violation: ${message} (Attempts left: ${remaining})`;
            }
            console.warn(`SECURITY VIOLATION: ${message}`);
        }

        // --- Progress Bar & Cancel Functions ---
        function stopProgress() {
            if (progressInterval) {
                clearInterval(progressInterval);
                progressInterval = null;
            }
            progressContainer.style.display = 'none';
            progressBarFill.style.width = '0%';
            progressText.textContent = '0%';
            dropArea.style.pointerEvents = 'auto'; 
        }

        function startProgress() {
            stopProgress(); 
            isCancelled = false;
            progressContainer.style.display = 'flex';
            dropArea.style.pointerEvents = 'none'; 
            
            let progress = 0;
            const duration = 30000; 
            const intervalTime = 100; 
            const step = (100 * intervalTime) / duration;

            progressInterval = setInterval(() => {
                if (isCancelled) {
                    stopProgress();
                    document.getElementById('messageArea').textContent = "❌ Order cancelled by user. Resetting...";
                    return;
                }
                
                if (progress < 99) {
                    progress += step;
                    progressBarFill.style.width = `${progress}%`;
                    progressText.textContent = `${Math.floor(progress)}%`;
                }
            }, intervalTime);
        }

        function cancelOrder() {
            isCancelled = true;
            currentFile = null;
            currentZip = null;
            document.getElementById('outputArea').style.display = 'none';
            stopProgress();
        }

        // --- Utility Functions (FIXED: Save/Copy functions added back) ---
        
        function arrayBufferToHex(buffer) {
            const view = new Uint8Array(buffer);
            let hexString = '';
            for (let i = 0; i < view.length; i++) {
                hexString += view[i].toString(16).padStart(2, '0') + ' ';
                if ((i + 1) % 32 === 0) { hexString += '\n'; }
            }
            return hexString.trim();
        }

        function readFileAs(file, method) {
            return new Promise((resolve, reject) => {
                const reader = new FileReader();
                reader.onload = (e) => resolve(e.target.result);
                reader.onerror = (e) => reject(e);
                
                if (method === 'arrayBuffer') { reader.readAsArrayBuffer(file); } 
                else if (method === 'text') { reader.readAsText(file); } 
                else if (method === 'dataURL') { reader.readAsDataURL(file); }
            });
        }
        
        function showTab(tabId) {
            document.querySelectorAll('.tab-content').forEach(el => el.style.display = 'none');
            document.querySelectorAll('.tab-button').forEach(btn => btn.classList.remove('active'));
            document.getElementById(tabId).style.display = 'block';
            document.querySelector(`button[onclick="showTab('${tabId}')"]`).classList.add('active');
        }

        // --- FIX: Copy function ---
        function copyContent(elementId) {
            const content = document.getElementById(elementId).textContent;
            navigator.clipboard.writeText(content).then(() => {
                document.getElementById('messageArea').textContent = "✅ Content copied successfully!";
                setTimeout(() => document.getElementById('messageArea').textContent = "", 3000);
            }).catch(err => {
                document.getElementById('messageArea').textContent = "❌ Copy failed (check permissions).";
            });
        }
        
        // --- FIX: Core Save function ---
        function saveAs(blob, fileName) {
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = fileName;
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
        }

        // Save Original File button handler
        function saveNormalContent() {
            if (!currentFile) {
                document.getElementById('messageArea').textContent = "⚠️ No current file to save.";
                return;
            }
            saveAs(currentFile, `downloaded_${currentFile.name}`);
            document.getElementById('messageArea').textContent = `✅ Original file saved as downloaded_${currentFile.name}!`;
        }

        // Save All Archive Content button handler
        async function saveAllZipContent() {
             document.getElementById('messageArea').textContent = "🛠️ Preparing archive contents...";
             if (!currentFile || !currentZip) {
                 document.getElementById('messageArea').textContent = "❌ No analyzed archive available.";
                 return;
             }
             
             try {
                const outputZip = await currentZip.generateAsync({ type: "blob" });
                const fileName = currentFile.name.replace(/\.[^/.]+$/, "") + '_extracted_contents.zip';
                
                saveAs(outputZip, fileName);
                document.getElementById('messageArea').textContent = `✅ Extracted content saved to ${fileName}!`;
             } catch (e) {
                 document.getElementById('messageArea').textContent = `❌ Failed to create save file: May be encrypted or corrupted.`;
             }
        }

        // --- Archive Analysis Functions (Unchanged) ---
        async function viewZipFileContent(relativePath) {
            if (!currentZip) return;
            
            try {
                document.getElementById('textContent').textContent = `Reading file: ${relativePath}...`;
                showTab('text-view'); 
                
                const zipEntry = currentZip.files[relativePath];
                if (zipEntry.dir) {
                    document.getElementById('textContent').textContent = `[Folder] ${relativePath} - Folder contents cannot be displayed directly.`;
                    return;
                }
                
                const content = await zipEntry.async('text'); 
                
                document.getElementById('textContent').textContent = content;
                document.getElementById('messageArea').textContent = `✅ Content displayed for: ${relativePath}`;

            } catch (e) {
                document.getElementById('textContent').textContent = `❌ Failed to display file (${relativePath}). It might be a binary file (.dex, .png, .mp3) or compressed XML.`;
                document.getElementById('messageArea').textContent = "⚠️ File is binary or corrupted. Cannot be read as text.";
            }
        }

        async function extractZipContent(buffer) {
            try {
                currentZip = await JSZip.loadAsync(buffer);
                zipContentList.innerHTML = '';
                
                currentZip.forEach((relativePath, zipEntry) => {
                    const listItem = document.createElement('li');
                    
                    const fileNameSpan = document.createElement('span');
                    fileNameSpan.className = 'file-name';
                    fileNameSpan.textContent = relativePath;

                    const fileSizeSpan = document.createElement('span');
                    fileSizeSpan.className = 'file-size';
                    fileSizeSpan.textContent = `(${(zipEntry.uncompressedSize / 1024).toFixed(2)} KB)`;

                    listItem.appendChild(fileNameSpan);
                    listItem.appendChild(fileSizeSpan);
                    
                    if (!zipEntry.dir) {
                        listItem.onclick = () => viewZipFileContent(relativePath);
                    } else {
                        listItem.style.backgroundColor = '#f7f7f7'; 
                        listItem.style.cursor = 'default';
                    }
                    
                    zipContentList.appendChild(listItem);
                });
                
            } catch (e) {
                zipContentList.innerHTML = `<li>❌ Failed to decompress: File might be encrypted or corrupted.</li>`;
            }
        }
        
        // --- Main Crushing Logic (Optimized for performance) ---
        async function crushFile(file) {
            if (checkBanStatus()) return;
            if (isCancelled) return; 

            currentFile = file;
            document.getElementById('messageArea').textContent = "⚙️ Starting analysis and crushing kernel...";
            zipTab.style.display = 'none'; 
            currentZip = null;
            startProgress(); 

            try {
                // 1. Security Check: Image Size Limit
                if (file.type.startsWith('image/') && file.size > MAX_IMAGE_SIZE_BYTES) {
                    recordViolation(`Image too detailed: Exceeded max size (${(file.size / (1024 * 1024)).toFixed(2)}MB > 5MB).`);
                    return;
                }
                
                // 2. Security Check: Code/Archive Size Limit (4 Million Tokens)
                const zipExtensions = ['.zip', '.aab', '.apk', '.jar']; 
                const isCodeArchive = zipExtensions.some(ext => file.name.toLowerCase().endsWith(ext));

                if (isCodeArchive && file.size > MAX_APK_SIZE_BYTES) {
                    recordViolation(`Overly complex code archive: Exceeded ~4 million tokens limit (${(file.size / (1024 * 1024)).toFixed(2)}MB > 16MB).`);
                    return;
                }
                
                // 3. Core Analysis (Using Promises to avoid blocking)
                const arrayBufferPromise = readFileAs(file, 'arrayBuffer');
                const textContentPromise = readFileAs(file, 'text');
                
                const [arrayBuffer, textContent] = await Promise.all([arrayBufferPromise, textContentPromise]);

                if (isCancelled) return; 

                document.getElementById('rawContent').textContent = arrayBufferToHex(arrayBuffer);
                document.getElementById('textContent').textContent = textContent;

                // 4. Normal View (Image/Audio)
                const normalContent = document.getElementById('normalContent');
                normalContent.innerHTML = '';
                if (file.type.startsWith('image/') || file.type.startsWith('audio/')) {
                    const url = URL.createObjectURL(file);
                    if (file.type.startsWith('image/')) {
                        normalContent.innerHTML = `<img src="${url}" style="max-width: 100%; max-height: 400px; display: block; margin: 10px auto; border-radius: 8px;">`;
                    } else if (file.type.startsWith('audio/')) {
                        normalContent.innerHTML = `<audio controls src="${url}"></audio>`;
                    }
                } else {
                     normalContent.innerHTML = `<p style="text-align: center; color: #7f8c8d;">No direct view available for this file type.</p>`;
                }
                
                // 5. Process Archives
                if (isCodeArchive) {
                     zipTab.style.display = 'block';
                     zipContentList.innerHTML = '<li>Decompressing and listing contents...</li>';
                     await extractZipContent(arrayBuffer); 
                }
                
                if (isCancelled) return;

                // Final success cleanup
                localStorage.setItem('fc_violations', '0'); 
                violationCount = 0;
                document.getElementById('messageArea').textContent = "✅ Analysis Complete! (Security compliant).";
                stopProgress(); 

            } catch (e) {
                if (isCancelled) return;
                document.getElementById('messageArea').textContent = `❌ File processing error: ${e.message}`;
                recordViolation(`Technical processing failure: ${e.message}`);
                stopProgress();
                console.error("Crush Error:", e);
            }
        }

        // --- Input Handling (Unchanged) ---
        function handleFiles(files) {
            if (files.length === 0) return;
            const file = files[0];
            
            // Update File Info
            document.getElementById('fileName').textContent = file.name;
            document.getElementById('fileSize').textContent = (file.size / (1024 * 1024)).toFixed(2) + ' MB';
            document.getElementById('fileType').textContent = file.type || 'Binary/Unknown';
            
            outputArea.style.display = 'block';
            showTab('normal-view');
            
            crushFile(file);
        }

        // Event Listeners
        dropArea.addEventListener('click', () => fileInput.click());
        fileInput.addEventListener('change', (e) => handleFiles(e.target.files));
        
        ['dragenter', 'dragover', 'dragleave', 'drop'].forEach(eventName => {
            dropArea.addEventListener(eventName, preventDefaults, false);
            document.body.addEventListener(eventName, preventDefaults, false);
        });
        function preventDefaults(e) { e.preventDefault(); e.stopPropagation(); }
        dropArea.addEventListener('drop', (e) => { handleFiles(e.dataTransfer.files); }, false);

        // --- Final Check on load ---
        checkBanStatus();

    </script>
</body>
</html>


