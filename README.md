<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Excel File Search</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.17.0/xlsx.full.min.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            padding: 0;
            background-color: #f4f4f9;
        }
        h1 {
            text-align: center;
            color: #333;
        }
        .container {
            max-width: 600px;
            margin: auto;
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }
        input[type="text"] {
            width: calc(100% - 22px);
            padding: 10px;
            margin-bottom: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }
        button {
            width: 100%;
            padding: 10px;
            border: none;
            background-color: #007bff;
            color: white;
            border-radius: 4px;
            cursor: pointer;
        }
        button:hover {
            background-color: #0056b3;
        }
        pre {
            background: #f8f8f8;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
            white-space: pre-wrap;
            word-wrap: break-word;
        }
        .log {
            color: red;
        }
    </style>
</head>
<body>
    <h1>Excel File Search</h1>
    <div class="container">
        <input type="text" id="searchQuery" placeholder="Enter search term">
        <button onclick="searchExcel()">Search</button>
        <pre id="results"></pre>
        <pre id="log" class="log"></pre>
    </div>

    <script>
        const repo = 'your_github_username/your_repo_name'; // GitHub 리포지토리
        const path = 'im.xlsx'; // 엑셀 파일 경로
        const token = 'your_github_token'; // GitHub Personal Access Token

        async function fetchExcelFromGithub() {
            const url = `https://api.github.com/repos/${repo}/contents/${path}`;
            const response = await fetch(url, {
                headers: {
                    'Authorization': `token ${token}`
                }
            });
            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }
            const data = await response.json();
            document.getElementById('log').textContent = 'API call successful';
            const fileContent = atob(data.content);
            const byteArray = new Uint8Array(fileContent.length);
            for (let i = 0; i < fileContent.length; i++) {
                byteArray[i] = fileContent.charCodeAt(i);
            }
            return byteArray.buffer;
        }

        async function searchExcel() {
            try {
                const searchQuery = document.getElementById('searchQuery').value.toLowerCase();
                const arrayBuffer = await fetchExcelFromGithub();
                const workbook = XLSX.read(new Uint8Array(arrayBuffer), {type: 'array'});
                const firstSheetName = workbook.SheetNames[0];
                const worksheet = workbook.Sheets[firstSheetName];
                const json = XLSX.utils.sheet_to_json(worksheet, { header: 1 });

                const results = json.filter(row => row[0] && row[0].toString().toLowerCase().includes(searchQuery));
                document.getElementById('results').textContent = JSON.stringify(results, null, 2);
                document.getElementById('log').textContent += '\nSearch completed';
            } catch (error) {
                document.getElementById('log').textContent = `Error: ${error.message}`;
            }
        }
    </script>
</body>
</html>
