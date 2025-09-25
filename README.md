# AI-problem-solver
Solution solver
<!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Problem Solver (Multilingual)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
    </style>
</head>
<body class="bg-gray-100 flex items-center justify-center min-h-screen p-4">

    <div class="w-full max-w-2xl bg-white rounded-xl shadow-2xl p-8 md:p-12">
        <h1 class="text-3xl md:text-4xl font-bold text-center text-gray-900 mb-6">
            समस्या हल करने वाला AI
        </h1>
        <p class="text-center text-gray-600 mb-8">
            यहाँ अपनी समस्या लिखें, और AI आपको इसे हल करने का रास्ता दिखाएगा।
        </p>

        <div class="space-y-6">
            <div>
                <label for="problem-input" class="block text-sm font-medium text-gray-700 mb-2">
                    आपकी समस्या क्या है? (हिंदी या English में)
                </label>
                <textarea id="problem-input" rows="4" class="block w-full px-4 py-3 rounded-lg border-2 border-gray-300 focus:ring-blue-500 focus:border-blue-500 transition-all shadow-sm" placeholder="जैसे: 'मुझे नई नौकरी ढूंढने में दिक्कत हो रही है' or 'I am finding it difficult to focus on my studies'"></textarea>
            </div>

            <button id="solve-button" class="w-full py-3 px-4 bg-blue-600 text-white font-bold rounded-lg shadow-lg hover:bg-blue-700 transition-colors focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2">
                हल ढूंढें
            </button>
        </div>

        <div id="solution-section" class="mt-8 hidden">
            <h2 class="text-2xl font-bold text-gray-900 mb-4">
                सुझाव
            </h2>
            <div id="loading-indicator" class="flex justify-center items-center hidden">
                <div class="animate-spin rounded-full h-8 w-8 border-t-2 border-b-2 border-blue-500"></div>
                <span class="ml-4 text-gray-500">सोचा जा रहा है...</span>
            </div>
            <div id="solution-content">
                <div id="solution-image-container" class="mb-6 rounded-lg overflow-hidden shadow-lg hidden">
                    <img id="solution-image" src="" alt="Solution Image" class="w-full h-auto">
                </div>
                <div id="solution-output" class="p-6 bg-gray-50 rounded-lg shadow-inner text-gray-800 leading-relaxed">
                    <!-- Solution will be displayed here -->
                </div>
            </div>
        </div>
    </div>

    <script>
        const problemInput = document.getElementById('problem-input');
        const solveButton = document.getElementById('solve-button');
        const solutionSection = document.getElementById('solution-section');
        const solutionOutput = document.getElementById('solution-output');
        const loadingIndicator = document.getElementById('loading-indicator');
        const solutionImageContainer = document.getElementById('solution-image-container');
        const solutionImage = document.getElementById('solution-image');

        function showLoading(show) {
            if (show) {
                loadingIndicator.classList.remove('hidden');
                solutionOutput.innerHTML = '';
                solutionOutput.classList.remove('hidden');
                solutionImageContainer.classList.add('hidden');
            } else {
                loadingIndicator.classList.add('hidden');
                solutionOutput.classList.remove('hidden');
            }
        }

        solveButton.addEventListener('click', async () => {
            const problem = problemInput.value.trim();
            if (problem === '') {
                solutionOutput.innerHTML = '<p class="text-red-500">कृपया अपनी समस्या लिखें। Please write your problem.</p>';
                solutionSection.classList.remove('hidden');
                return;
            }

            solutionSection.classList.remove('hidden');
            showLoading(true);

            const apiKey = ""; 
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-image-preview:generateContent?key=${apiKey}`;

            const payload = {
                contents: [{ 
                    parts: [{ text: `मेरी समस्या को हल करने में मेरी मदद करें। समस्या है: "${problem}"। मुझे कुछ सुझाव दें जो मुझे इसे हल करने में मदद करें। सुझाव हिंदी या English में दें, जो भी भाषा मेरे प्रश्न के लिए सबसे उपयुक्त हो। सुझाव के साथ एक चित्र भी बनाएं जो समस्या से संबंधित हो।` }] 
                }],
                generationConfig: {
                    responseModalities: ['TEXT', 'IMAGE']
                }
            };

            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }

                const result = await response.json();
                const textPart = result.candidates?.[0]?.content?.parts?.find(p => p.text);
                const imagePart = result.candidates?.[0]?.content?.parts?.find(p => p.inlineData);

                if (textPart) {
                    solutionOutput.innerHTML = textPart.text.replace(/\n/g, '<br>');
                } else {
                    solutionOutput.innerHTML = '<p class="text-red-500">कोई सुझाव नहीं मिला। कृपया अपनी समस्या को और स्पष्ट करें। No solution found. Please make your problem clearer.</p>';
                }

                if (imagePart) {
                    solutionImage.src = `data:${imagePart.inlineData.mimeType};base64,${imagePart.inlineData.data}`;
                    solutionImageContainer.classList.remove('hidden');
                } else {
                    solutionImageContainer.classList.add('hidden');
                }

            } catch (error) {
                console.error("AI se jawab lene mein error:", error);
                solutionOutput.innerHTML = '<p class="text-red-500">कुछ गड़बड़ी हुई। कृपया थोड़ी देर बाद फिर से कोशिश करें। Something went wrong. Please try again later.</p>';
            } finally {
                showLoading(false);
            }
        });
    </script>

</body>
</html>
