<html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Juego: Unscramble the Verb</title>
    <!-- Chosen Palette: Warm Harmony - A calming palette using a light neutral background (#F0F2F5), white content areas, a soft blue (#88A0B2) for primary interactive elements, and subtle, harmonious accent colors for feedback and controls. -->
    <!-- Application Structure Plan: The application is designed as a single-view game interface to maintain user focus. The structure is vertical: Title, Score, Image Hint, Scrambled Letters, Answer Slots, Feedback, and Controls. This linear flow guides the user from the puzzle (image/letters) to the interaction area (slots) and finally to the action buttons (check/next). This was chosen for its simplicity and intuitive nature, making the game immediately understandable without instructions. -->
    <!-- Visualization & Content Choices: Report Info: Verbs list (word, scrambled, image). Goal: Teach verbs. Viz/Presentation: Interactive HTML blocks for letters/slots. Interaction: Click-to-move letters for simplicity. Justification: This direct manipulation is intuitive on both desktop and touch devices. Data (verbs) is stored in a JS array for easy management. Feedback is multi-sensory (color, text, sound, animation) to reinforce learning. Library/Method: Vanilla JS for all logic, Tailwind CSS for styling. -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700;800&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #F0F2F5;
        }
        .letter-block, .letter-slot {
            display: flex;
            justify-content: center;
            align-items: center;
            width: 50px;
            height: 50px;
            font-size: 1.75rem;
            font-weight: bold;
            border-radius: 0.5rem;
            cursor: pointer;
            user-select: none;
            transition: all 0.2s ease-in-out;
        }
        .letter-block {
            background-color: #88A0B2;
            color: white;
            box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
        }
        .letter-block:hover {
            transform: translateY(-2px) scale(1.05);
            background-color: #708a9e;
        }
        .letter-slot {
            background-color: #E5E7EB;
            border: 2px dashed #9CA3AF;
            color: #374151;
        }
        .letter-slot.filled {
            background-color: #4B5563;
            color: white;
            border-style: solid;
            transform: scale(1.05);
        }
        .feedback-message {
            transition: opacity 0.3s, transform 0.3s;
        }
        @keyframes pop-in {
            0% { transform: scale(0.9); opacity: 0; }
            50% { transform: scale(1.05); opacity: 1; }
            100% { transform: scale(1); opacity: 1; }
        }
        .animate-pop { animation: pop-in 0.4s ease-out forwards; }
        @keyframes shake {
            0%, 100% { transform: translateX(0); }
            10%, 30%, 50%, 70%, 90% { transform: translateX(-6px); }
            20%, 40%, 60%, 80% { transform: translateX(6px); }
        }
        .animate-shake { animation: shake 0.4s ease-in-out; }
        .modal {
            display: none;
            justify-content: center;
            align-items: center;
            position: fixed; /* Ensures the modal is always visible */
            inset: 0;
            background-color: rgba(0, 0, 0, 0.5); /* Semi-transparent background */
            z-index: 1000; /* Ensures it's above other elements */
        }
        .modal.active { display: flex; }
        .modal-content {
            background-color: white;
            padding: 2rem;
            border-radius: 0.75rem;
            box-shadow: 0 10px 15px rgba(0, 0, 0, 0.1), 0 4px 6px rgba(0, 0, 0, 0.05);
            text-align: center;
            max-width: 90%;
            width: 400px; /* Fixed width for modal content */
        }
        .loading-spinner {
            border: 4px solid rgba(0, 0, 0, 0.1);
            border-left-color: #88A0B2;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
            margin-top: 20px;
            display: none; /* Hidden by default */
        }
        @keyframes spin {
            to {
                transform: rotate(360deg);
            }
        }
    </style>
</head>
<body class="flex items-center justify-center min-h-screen p-4">

    <div id="game-container" class="bg-white p-6 sm:p-8 rounded-2xl shadow-2xl max-w-md w-full flex flex-col items-center gap-5">
        <h1 class="text-3xl sm:text-4xl font-extrabold text-gray-800 text-center">Unscramble the Verb</h1>
        
        <div class="text-2xl font-bold text-gray-700 bg-gray-100 px-4 py-2 rounded-lg">
            Puntuación: <span id="scoreDisplay">0</span>
        </div>

        <div class="w-full h-48 bg-gray-200 rounded-lg overflow-hidden shadow-inner flex items-center justify-center relative">
            <img id="verbImage" src="" alt="Pista visual del verbo" class="object-contain w-full h-full">
            <div id="imageLoadingSpinner" class="loading-spinner absolute"></div>
        </div>

        <p class="text-gray-600 text-center">Coloca las letras en el orden correcto.</p>

        <div id="wordSlots" class="flex flex-wrap justify-center gap-2 sm:gap-3 p-2 min-h-[66px] w-full"></div>
        <div id="scrambledLetters" class="flex flex-wrap justify-center gap-2 sm:gap-3 p-3 bg-gray-100 rounded-lg w-full min-h-[66px]"></div>
        
        <div id="feedbackContainer" class="h-8">
            <p id="feedbackMessage" class="feedback-message text-xl font-semibold opacity-0"></p>
        </div>

        <div class="flex flex-col sm:flex-row gap-3 mt-2 w-full">
            <button id="resetButton" class="w-full sm:w-auto flex-1 bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-3 px-4 rounded-lg shadow-md transition duration-200 transform hover:scale-105">Reiniciar</button>
            <button id="checkButton" class="w-full sm:w-auto flex-1 bg-green-500 hover:bg-green-600 text-white font-bold py-3 px-4 rounded-lg shadow-md transition duration-200 transform hover:scale-105">Comprobar</button>
        </div>
        <button id="nextButton" class="w-full bg-blue-500 hover:bg-blue-600 text-white font-bold py-3 px-4 rounded-lg shadow-md transition duration-200 transform hover:scale-105 opacity-50 cursor-not-allowed" disabled>Siguiente</button>
    </div>

    <div id="modal" class="modal fixed inset-0 bg-black bg-opacity-50">
        <div class="bg-white p-8 rounded-lg shadow-xl text-center">
            <p id="modalMessage" class="text-lg mb-6"></p>
            <button id="modalConfirmBtn" class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-6 rounded-lg">OK</button>
        </div>
    </div>

    <script>
        // Array of game data: word, scrambled version, and prompt for image generation
        const verbs = [
            { word: "RUN", scrambled: "NUR", prompt: 'a cartoon child running happily, vibrant colors, clear background, isolated' },
            { word: "JUMP", scrambled: "MPUJ", prompt: 'a cartoon child jumping happily, vibrant colors, clear background, isolated' },
            { word: "EAT", scrambled: "TEA", prompt: 'a cartoon child eating a snack, vibrant colors, clear background, isolated' },
            { word: "READ", scrambled: "DARE", prompt: 'a cartoon child reading a book, vibrant colors, clear background, isolated' },
            { word: "PLAY", scrambled: "YAPL", prompt: 'a cartoon child playing with a toy, vibrant colors, clear background, isolated' },
            { word: "DANCE", scrambled: "ECNDA", prompt: 'a cartoon child dancing with joy, vibrant colors, clear background, isolated' },
            { word: "SLEEP", scrambled: "PLESE", prompt: 'a cartoon child sleeping peacefully, vibrant colors, clear background, isolated' }
        ];

        let currentVerbIndex = 0;
        let score = 0;
        let lettersState = { source: [], destination: [] };

        // Cache DOM elements for efficient access
        const DOMElements = {
            scoreDisplay: document.getElementById('scoreDisplay'),
            verbImage: document.getElementById('verbImage'),
            wordSlots: document.getElementById('wordSlots'),
            scrambledLetters: document.getElementById('scrambledLetters'),
            feedbackMessage: document.getElementById('feedbackMessage'),
            checkButton: document.getElementById('checkButton'),
            nextButton: document.getElementById('nextButton'),
            resetButton: document.getElementById('resetButton'),
            modal: document.getElementById('modal'),
            modalMessage: document.getElementById('modalMessage'),
            modalConfirmBtn: document.getElementById('modalConfirmBtn'),
            imageLoadingSpinner: document.getElementById('imageLoadingSpinner')
        };

        /**
         * Shuffles an array using the Fisher-Yates (Knuth) algorithm.
         * @param {Array} array - The array to shuffle.
         * @returns {Array} The shuffled array.
         */
        function shuffleArray(array) {
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]];
            }
            return array;
        }

        /**
         * Updates the displayed score on the UI.
         */
        function updateScoreDisplay() {
            DOMElements.scoreDisplay.textContent = score;
        }

        /**
         * Generates a simple success sound using the Web Audio API.
         */
        function playSuccessSound() {
            const audioContext = new (window.AudioContext || window.webkitAudioContext)();
            const oscillator = audioContext.createOscillator();
            const gainNode = audioContext.createGain();

            oscillator.connect(gainNode);
            gainNode.connect(audioContext.destination);

            oscillator.type = 'sine'; // A smooth wave
            oscillator.frequency.setValueAtTime(440, audioContext.currentTime); // A4 note
            gainNode.gain.setValueAtTime(0.5, audioContext.currentTime); // Volume

            oscillator.start();
            gainNode.gain.exponentialRampToValueAtTime(0.001, audioContext.currentTime + 0.3); // Fade out
            oscillator.stop(audioContext.currentTime + 0.3); // Stop after 0.3 seconds
        }

        /**
         * Generates a simple error sound using the Web Audio API.
         */
        function playErrorSound() {
            const audioContext = new (window.AudioContext || window.webkitAudioContext)();
            const oscillator = audioContext.createOscillator();
            const gainNode = audioContext.createGain();

            oscillator.connect(gainNode);
            gainNode.connect(audioContext.destination);

            oscillator.type = 'triangle'; // A harsher wave
            oscillator.frequency.setValueAtTime(220, audioContext.currentTime); // A3 note
            gainNode.gain.setValueAtTime(0.5, audioContext.currentTime); // Volume

            oscillator.start();
            gainNode.gain.exponentialRampToValueAtTime(0.001, audioContext.currentTime + 0.4); // Fade out
            oscillator.stop(audioContext.currentTime + 0.4); // Stop after 0.4 seconds
        }

        /**
         * Generates an image based on a given prompt using the Imagen API.
         * Displays a loading spinner while fetching the image.
         * @param {string} prompt - The text prompt for image generation.
         * @returns {Promise<string>} A promise that resolves to the base64 encoded image URL or a fallback URL.
         */
        async function generateImage(prompt) {
            // API key is automatically provided by the Canvas environment for Google models.
            const apiKey = ""; 
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/imagen-3.0-generate-002:predict?key=${apiKey}`;
            const payload = { instances: { prompt: prompt }, parameters: { "sampleCount": 1} };

            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });
                const result = await response.json();
                if (result.predictions && result.predictions.length > 0 && result.predictions[0].bytesBase64Encoded) {
                    return `data:image/png;base64,${result.predictions[0].bytesBase64Encoded}`;
                } else {
                    console.error("Error generating image: No predictions found or bytesBase64Encoded missing.");
                    // Fallback image in case of API error or no valid response
                    return `https://placehold.co/400x300/CCCCCC/000000?text=Error+Loading`; 
                }
            } catch (error) {
                console.error("Error fetching image:", error);
                // Fallback image for network errors
                return `https://placehold.co/400x300/CCCCCC/000000?text=Error+Loading`; 
            }
        }

        /**
         * Loads images for all verbs in the `verbs` array.
         * Shows/hides a loading spinner and disables/enables game buttons during the process.
         */
        async function loadAllVerbImages() {
            DOMElements.imageLoadingSpinner.style.display = 'block'; // Show spinner
            DOMElements.verbImage.style.display = 'none'; // Hide image while loading
            
            // Disable game buttons during loading to prevent interaction before images are ready
            DOMElements.checkButton.disabled = true;
            DOMElements.nextButton.disabled = true;
            DOMElements.resetButton.disabled = true;
            
            // Generate and store image URLs for each verb
            for (let i = 0; i < verbs.length; i++) {
                verbs[i].image = await generateImage(verbs[i].prompt);
            }

            DOMElements.imageLoadingSpinner.style.display = 'none'; // Hide spinner
            DOMElements.verbImage.style.display = 'block'; // Show image
            
            // Enable game buttons after images are loaded
            DOMElements.checkButton.disabled = false;
            // Next button's state will be further managed by initializeGame based on completion
            DOMElements.nextButton.disabled = false; 
            DOMElements.resetButton.disabled = false;
            
            initializeGame(); // Initialize the game after images are loaded
        }

        /**
         * Initializes the game for the current verb.
         * Sets up the scrambled letters and empty word slots.
         */
        function initializeGame() {
            const verbData = verbs[currentVerbIndex];
            // Populate source letters with shuffled scrambled word
            lettersState.source = shuffleArray([...verbData.scrambled]);
            // Create empty slots for the destination word
            lettersState.destination = Array(verbData.word.length).fill(null);
            
            DOMElements.verbImage.src = verbData.image;
            DOMElements.verbImage.alt = `Pista para el verbo ${verbData.word}`;
            
            updateScoreDisplay();
            updateUI(); // Render the letters and slots on the UI
            resetFeedback(); // Clear any previous feedback messages
            
            DOMElements.checkButton.disabled = false; // Re-enable check button for new word
            // Disable the next button until the current word is correctly guessed
            DOMElements.nextButton.disabled = true; 
            DOMElements.nextButton.classList.add('opacity-50', 'cursor-not-allowed');
            DOMElements.nextButton.classList.remove('animate-pulse'); // Remove pulse animation
        }

        /**
         * Updates the UI to reflect the current state of `lettersState`.
         * Renders scrambled letters and word slots.
         */
        function updateUI() {
            // Clear existing letters and re-render scrambled letters
            DOMElements.scrambledLetters.innerHTML = '';
            lettersState.source.forEach((letter, index) => {
                if (letter) { // Only render if there's a letter at this position
                    DOMElements.scrambledLetters.appendChild(createLetterBlock(letter, 'source', index));
                }
            });

            // Clear existing slots and re-render word slots
            DOMElements.wordSlots.innerHTML = '';
            lettersState.destination.forEach((letter, index) => {
                const slot = createLetterBlock(letter, 'destination', index);
                if (letter) { // If a slot is filled, add 'filled' class
                    slot.classList.add('filled');
                }
                DOMElements.wordSlots.appendChild(slot);
            });
        }

        /**
         * Creates a clickable letter block or slot element.
         * @param {string|null} letter - The letter to display, or null for an empty slot.
         * @param {'source'|'destination'} type - The type of element ('source' for scrambled letters, 'destination' for word slots).
         * @param {number} index - The index of the letter/slot in its respective array.
         * @returns {HTMLElement} The created div element.
         */
        function createLetterBlock(letter, type, index) {
            const el = document.createElement('div');
            el.textContent = letter; // Display the letter
            
            if (type === 'destination') {
                el.classList.add('letter-slot'); // Apply slot-specific styling
                if (letter) { // Only filled slots are clickable to move letters back
                    el.addEventListener('click', () => moveLetter(index, 'destination', 'source'));
                }
            } else { // 'source' type
                el.classList.add('letter-block'); // Apply letter block styling
                el.addEventListener('click', () => moveLetter(index, 'source', 'destination'));
            }
            return el;
        }

        /**
         * Moves a letter from one list (source/destination) to another.
         * @param {number} fromIndex - The index of the letter to move.
         * @param {'source'|'destination'} fromList - The list from which to move the letter.
         * @param {'source'|'destination'} toList - The list to which to move the letter.
         */
        function moveLetter(fromIndex, fromList, toList) {
            const letter = lettersState[fromList][fromIndex];
            if (!letter) return; // Do nothing if there's no letter to move

            // Find the first empty slot in the destination list
            const toIndex = lettersState[toList].indexOf(null);
            if (toIndex !== -1) { // If an empty slot is found
                lettersState[toList][toIndex] = letter; // Move the letter
                lettersState[fromList][fromIndex] = null; // Clear the original position
                updateUI(); // Re-render UI after movement
            }
        }

        /**
         * Checks the user's entered word against the correct verb.
         * Provides feedback and updates score.
         */
        function checkAnswer() {
            const enteredWord = lettersState.destination.join('');
            // Check if all slots are filled
            if (enteredWord.length !== verbs[currentVerbIndex].word.length) {
                showModal("¡Por favor, completa la palabra!"); // Custom modal for incomplete word
                return;
            }

            if (enteredWord === verbs[currentVerbIndex].word) {
                score++; // Increment score for correct answer
                showFeedback(true); // Show positive feedback
                DOMElements.checkButton.disabled = true; // Disable check button after correct answer
                DOMElements.nextButton.disabled = false; // Enable next button
                DOMElements.nextButton.classList.remove('opacity-50', 'cursor-not-allowed');
                DOMElements.nextButton.classList.add('animate-pulse'); // Add pulse animation to next button
            } else {
                score = Math.max(0, score - 1); // Decrement score, but not below 0
                showFeedback(false); // Show negative feedback
                DOMElements.wordSlots.classList.add('animate-shake'); // Add shake animation to slots
            }
            updateScoreDisplay(); // Update score display
        }

        /**
         * Displays visual and audio feedback to the user.
         * @param {boolean} isCorrect - True if the answer is correct, false otherwise.
         */
        function showFeedback(isCorrect) {
            resetFeedback(); // Clear previous feedback before showing new one
            setTimeout(() => { // Small delay to allow CSS transitions to reset
                const feedback = DOMElements.feedbackMessage;
                if (isCorrect) {
                    feedback.textContent = "¡Correcto!";
                    feedback.classList.add('correct', 'text-green-600'); // Green text for correct
                    playSuccessSound(); // Play success sound using Web Audio API
                } else {
                    feedback.textContent = "Inténtalo de nuevo";
                    feedback.classList.add('incorrect', 'text-red-600'); // Red text for incorrect
                    playErrorSound(); // Play error sound using Web Audio API
                }
                feedback.classList.add('animate-pop'); // Apply pop-in animation
                feedback.classList.remove('opacity-0'); // Make feedback visible
            }, 10);
        }

        /**
         * Resets the feedback message and removes animations.
         */
        function resetFeedback() {
            const feedback = DOMElements.feedbackMessage;
            feedback.textContent = '';
            // Reset all feedback-related classes
            feedback.className = 'feedback-message text-xl font-semibold opacity-0';
            DOMElements.wordSlots.classList.remove('animate-shake'); // Remove shake animation
        }

        /**
         * Advances to the next verb in the list or finishes the game if all verbs are completed.
         */
        function nextVerb() {
            currentVerbIndex++;
            if (currentVerbIndex < verbs.length) {
                initializeGame(); // Load the next verb
            } else {
                finishGame(); // All verbs completed
            }
        }

        /**
         * Handles game completion, disabling buttons and showing a final score modal.
         */
        function finishGame() {
            DOMElements.checkButton.disabled = true;
            DOMElements.nextButton.disabled = true;
            DOMElements.nextButton.classList.add('opacity-50', 'cursor-not-allowed');
            DOMElements.nextButton.classList.remove('animate-pulse');
            DOMElements.resetButton.disabled = false; // Allow restarting the game from the modal

            showModal(`¡Juego Completado! Tu puntuación final es: ${score} puntos.`, true); // Pass true for game completion
        }

        /**
         * Displays a custom modal with a message.
         * @param {string} message - The message to display in the modal.
         * @param {boolean} [isGameCompleted=false] - True if the modal is for game completion, changes button text.
         */
        function showModal(message, isGameCompleted = false) {
            DOMElements.modalMessage.textContent = message;
            DOMElements.modal.classList.add('active'); // Show the modal
            if (isGameCompleted) {
                DOMElements.modalConfirmBtn.textContent = "Volver a Jugar"; // Change button text for game over
                DOMElements.modalConfirmBtn.onclick = () => {
                    DOMElements.modal.classList.remove('active'); // Hide modal
                    currentVerbIndex = 0; // Reset index to start a new game
                    score = 0; // Reset score for a new game
                    loadAllVerbImages(); // Re-load images and start a new game
                };
            } else {
                DOMElements.modalConfirmBtn.textContent = "OK"; // Default button text
                DOMElements.modalConfirmBtn.onclick = () => {
                    DOMElements.modal.classList.remove('active'); // Hide modal
                };
            }
        }

        // --- Event Listeners ---
        // Confirm button in modal to close it
        DOMElements.modalConfirmBtn.addEventListener('click', () => {
            DOMElements.modal.classList.remove('active');
        });
        // Check button to verify the answer
        DOMElements.checkButton.addEventListener('click', checkAnswer);
        // Next button to load the next verb
        DOMElements.nextButton.addEventListener('click', nextVerb);
        // Reset button to restart the current word
        DOMElements.resetButton.addEventListener('click', () => {
            initializeGame(); // Re-initialize the current game state
        });

        // Start the game by loading all images when the window loads
        window.onload = loadAllVerbImages;
    </script>
</body>
</html>
