<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GitHub File Browser</title>
    <!-- Use Tailwind CSS for a modern, clean design -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6;
        }
        .container {
            max-width: 768px;
        }
        .file-item {
            display: flex;
            align-items: center;
            padding: 0.75rem 1rem;
            border-bottom: 1px solid #e5e7eb;
            transition: all 0.2s ease-in-out;
            cursor: pointer;
        }
        .file-item:hover {
            background-color: #e5e7eb;
        }
        .icon {
            width: 24px;
            height: 24px;
            margin-right: 1rem;
        }
    </style>
</head>
<body class="p-8">

    <div class="container mx-auto bg-white rounded-xl shadow-lg p-6">
        <h1 class="text-3xl font-bold mb-4 text-gray-800">Repository Contents</h1>
        <p class="text-gray-500 mb-6">
            Displaying contents of the <code class="bg-gray-200 px-1 py-0.5 rounded text-sm">src</code> folder.
            To change the folder, edit the `GITHUB_API_URL` variable in the script below.
        </p>

        <div id="file-list" class="flex flex-col">
            <!-- Loading message or file list will be inserted here by JavaScript -->
            <div class="text-center text-gray-400 py-8">
                <svg class="animate-spin h-8 w-8 text-gray-400 mx-auto mb-2" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                    <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                    <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                </svg>
                <p>Loading files from GitHub...</p>
            </div>
        </div>
    </div>

    <script>
        // --- CONFIGURATION SECTION ---
        // IMPORTANT: You MUST update these variables to match your repository.
        const GITHUB_USERNAME = "DennisMire"; // e.g., "octocat"
        const REPOSITORY_NAME = "test2.github.io"; // e.g., "Spoon-Knife"
        const FOLDER_PATH = "src"; // e.g., "src" or "assets/images"

        const GITHUB_API_URL = `https://api.github.com/repos/${GITHUB_USERNAME}/${REPOSITORY_NAME}/contents/${FOLDER_PATH}`;

        // Get the DOM elements
        const fileListContainer = document.getElementById('file-list');

        // Function to create an icon based on the file type
        function getIconSvg(type) {
            if (type === 'dir') {
                return `<svg class="icon text-blue-500" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M4 20h16a2 2 0 0 0 2-2V8a2 2 0 0 0-2-2h-8l-2-3H4a2 2 0 0 0-2 2v11a2 2 0 0 0 2 2z"></path></svg>`;
            } else {
                return `<svg class="icon text-gray-500" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path><polyline points="14 2 14 8 20 8"></polyline><line x1="16" y1="13" x2="8" y2="13"></line><line x1="16" y1="17" x2="8" y2="17"></line><line x1="10" y1="9" x2="8" y2="9"></line></svg>`;
            }
        }

        // Main function to fetch and display the contents
        async function fetchRepoContents() {
            // Check for placeholder values before attempting the API call
            if (GITHUB_USERNAME === "YOUR-USERNAME" || REPOSITORY_NAME === "YOUR-REPOSITORY") {
                fileListContainer.innerHTML = `
                    <div class="text-center bg-yellow-100 border border-yellow-300 rounded-lg p-6">
                        <p class="text-lg font-bold text-yellow-800 mb-2">Configuration Error</p>
                        <p class="text-yellow-700">Please open the HTML file and update the <code class="bg-yellow-200 px-1 py-0.5 rounded text-sm">GITHUB_USERNAME</code> and <code class="bg-yellow-200 px-1 py-0.5 rounded text-sm">REPOSITORY_NAME</code> variables with your repository details.</p>
                    </div>
                `;
                return; // Stop execution
            }

            try {
                const response = await fetch(GITHUB_API_URL);

                if (!response.ok) {
                    throw new Error(`GitHub API returned an error: ${response.status} ${response.statusText}`);
                }

                const data = await response.json();

                // Clear the loading message
                fileListContainer.innerHTML = '';

                // Check if the directory is empty
                if (data.length === 0) {
                    fileListContainer.innerHTML = `<div class="text-center text-gray-400 py-8">This folder is empty.</div>`;
                    return;
                }

                // Sort folders before files
                data.sort((a, b) => {
                    if (a.type === 'dir' && b.type !== 'dir') return -1;
                    if (a.type !== 'dir' && b.type === 'dir') return 1;
                    return 0;
                });

                data.forEach(item => {
                    const itemDiv = document.createElement('a');
                    itemDiv.href = item.html_url; // Link to the file/folder on GitHub
                    itemDiv.target = "_blank"; // Open in a new tab
                    itemDiv.className = 'file-item rounded-lg mb-2 shadow-sm';
                    
                    itemDiv.innerHTML = `
                        ${getIconSvg(item.type)}
                        <span class="text-lg font-medium text-gray-700">${item.name}</span>
                    `;

                    fileListContainer.appendChild(itemDiv);
                });

            } catch (error) {
                console.error("Failed to fetch repository contents:", error);
                fileListContainer.innerHTML = `
                    <div class="text-center text-red-500 py-8">
                        <p class="mb-2">Error: Failed to load repository contents.</p>
                        <p class="text-sm">Please check the console for details and ensure you have updated the GITHUB_API_URL in the code.</p>
                    </div>
                `;
            }
        }

        // Call the function when the page loads
        window.onload = fetchRepoContents;
    </script>
</body>
</html>
