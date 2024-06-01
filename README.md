<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Excel File Search</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.17.0/xlsx.full.min.js"></script>
</head>
<body>
    <h1>Excel File Search</h1>
    <input type="text" id="searchQuery" placeholder="Enter search term">
    <button onclick="searchExcel()">Search</button>
    <pre id="results"></pre>

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
            const data = await response.json();
            const fileContent = atob(data.content);
            const byteArray = new Uint8Array(fileContent.length);
            for (let i = 0; i < fileContent.length; i++) {
                byteArray[i] = fileContent.charCodeAt(i);
            }
            return byteArray.buffer;
        }

        async function searchExcel() {
            const searchQuery = document.getElementById('searchQuery').value.toLowerCase();
            const arrayBuffer = await fetchExcelFromGithub();
            const workbook = XLSX.read(new Uint8Array(arrayBuffer), {type: 'array'});
            const firstSheetName = workbook.SheetNames[0];
            const worksheet = workbook.Sheets[firstSheetName];
            const json = XLSX.utils.sheet_to_json(worksheet, { header: 1 });

            const results = json.filter(row => row[0] && row[0].toString().toLowerCase().includes(searchQuery));
            document.getElementById('results').textContent = JSON.stringify(results, null, 2);
        }
    </script>
</body>
</html>
