<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>iA aCademY - Login</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;800&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        .animate-fade-in {
            animation: fadeIn 0.5s ease-out;
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }
    </style>
</head>
<body class="min-h-screen font-sans text-white p-4 sm:p-8 flex flex-col items-center justify-center bg-gradient-to-br from-black to-green-900">

    <header class="w-full max-w-md text-center mb-8">
        <h1 class="text-5xl sm:text-6xl font-extrabold text-green-400 leading-tight mb-2">
            iA aCademY
        </h1>
        <p class="text-xl sm:text-2xl text-green-200">
            Acesse sua conta
        </p>
    </header>

    <main id="main-content" class="w-full max-w-md bg-gray-800 bg-opacity-75 shadow-xl rounded-3xl p-6 sm:p-10">
        <div id="loading-screen" class="text-center">
            <div class="text-2xl text-white">Carregando...</div>
        </div>

        <div id="auth-form" class="animate-fade-in hidden">
            <h2 id="form-title" class="text-3xl font-bold text-white mb-6 text-center">Login</h2>
            <div id="error-message" class="bg-red-900 border border-red-700 text-red-300 px-4 py-3 rounded-lg relative mb-4 hidden" role="alert">
                <span id="error-text" class="block sm:inline"></span>
            </div>
            <form id="auth-form-element" class="space-y-6">
                <div>
                    <label for="email" class="block text-lg font-medium text-gray-300 mb-2">
                        E-mail
                    </label>
                    <input
                        type="email"
                        id="email"
                        required
                        class="w-full px-5 py-3 border border-gray-600 rounded-xl shadow-sm focus:ring-green-500 focus:border-green-500 text-lg placeholder-gray-500 bg-gray-700 text-white"
                        placeholder="seu@email.com"
                    />
                </div>
                <div>
                    <label for="password" class="block text-lg font-medium text-gray-300 mb-2">
                        Senha
                    </label>
                    <input
                        type="password"
                        id="password"
                        required
                        class="w-full px-5 py-3 border border-gray-600 rounded-xl shadow-sm focus:ring-green-500 focus:border-green-500 text-lg placeholder-gray-500 bg-gray-700 text-white"
                        placeholder="********"
                    />
                </div>
                <div>
                    <button
                        type="submit"
                        id="submit-button"
                        class="w-full px-6 py-4 bg-green-600 text-white font-semibold rounded-full shadow-lg hover:bg-green-700 transition duration-300 ease-in-out transform hover:scale-105 focus:outline-none focus:ring-4 focus:ring-green-300 text-xl"
                    >
                        Entrar
                    </button>
                </div>
            </form>
            <div class="mt-6 text-center">
                <button
                    id="toggle-auth-mode"
                    class="text-green-400 hover:text-green-600 font-medium transition duration-300 ease-in-out text-lg"
                >
                    Não tem uma conta? Registre-se
                </button>
            </div>
        </div>

        <div id="protected-content" class="text-center animate-fade-in hidden">
            <h2 class="text-3xl font-bold text-white mb-4">Bem-vindo!</h2>
            <p class="text-lg text-gray-300 mb-6">
                Você está logado como: <span id="user-info" class="font-semibold text-green-400 break-all"></span>
            </p>
            <button
                id="logout-button"
                class="px-8 py-4 bg-red-600 text-white font-semibold rounded-full shadow-lg hover:bg-red-700 transition duration-300 ease-in-out transform hover:scale-105 focus:outline-none focus:ring-4 focus:ring-red-300"
            >
                Sair
            </button>
        </div>
    </main>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import {
            getAuth,
            signInAnonymously,
            signInWithCustomToken,
            onAuthStateChanged,
            createUserWithEmailAndPassword,
            signInWithEmailAndPassword,
            signOut
        } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Global variables for Firebase
        let app;
        let auth;
        let db;
        let currentUser = null;
        let isRegistering = false; // State to toggle between login and register forms

        // DOM elements
        const loadingScreen = document.getElementById('loading-screen');
        const authFormDiv = document.getElementById('auth-form');
        const protectedContentDiv = document.getElementById('protected-content');
        const formTitle = document.getElementById('form-title');
        const errorMessageDiv = document.getElementById('error-message');
        const errorText = document.getElementById('error-text');
        const emailInput = document.getElementById('email');
        const passwordInput = document.getElementById('password');
        const submitButton = document.getElementById('submit-button');
        const toggleAuthModeButton = document.getElementById('toggle-auth-mode');
        const userInfoSpan = document.getElementById('user-info');
        const logoutButton = document.getElementById('logout-button');
        const authFormElement = document.getElementById('auth-form-element');

        // Function to display errors
        function displayError(message) {
            errorText.textContent = message;
            errorMessageDiv.classList.remove('hidden');
        }

        // Function to hide errors
        function hideError() {
            errorMessageDiv.classList.add('hidden');
            errorText.textContent = '';
        }

        // Function to update UI based on authentication state
        function updateUI() {
            hideError();
            if (currentUser) {
                loadingScreen.classList.add('hidden');
                authFormDiv.classList.add('hidden');
                protectedContentDiv.classList.remove('hidden');
                userInfoSpan.textContent = currentUser.email || currentUser.uid;
            } else {
                loadingScreen.classList.add('hidden');
                authFormDiv.classList.remove('hidden');
                protectedContentDiv.classList.add('hidden');
                formTitle.textContent = isRegistering ? 'Registrar' : 'Login';
                submitButton.textContent = isRegistering ? 'Registrar' : 'Entrar';
                toggleAuthModeButton.textContent = isRegistering ? 'Já tem uma conta? Faça login' : 'Não tem uma conta? Registre-se';
                emailInput.value = '';
                passwordInput.value = '';
            }
        }

        // Handle user registration
        async function handleRegister(e) {
            e.preventDefault();
            hideError();
            if (!auth) {
                displayError("Firebase Auth não inicializado.");
                return;
            }
            try {
                await createUserWithEmailAndPassword(auth, emailInput.value, passwordInput.value);
                // User is automatically logged in after registration, onAuthStateChanged will handle UI update
            } catch (err) {
                console.error("Erro durante o registro:", err);
                displayError(err.message);
            }
        }

        // Handle user login
        async function handleLogin(e) {
            e.preventDefault();
            hideError();
            if (!auth) {
                displayError("Firebase Auth não inicializado.");
                return;
            }
            try {
                await signInWithEmailAndPassword(auth, emailInput.value, passwordInput.value);
                // User is automatically logged in after login, onAuthStateChanged will handle UI update
            } catch (err) {
                console.error("Erro durante o login:", err);
                displayError(err.message);
            }
        }

        // Handle user logout
        async function handleLogout() {
            hideError();
            if (!auth) {
                displayError("Firebase Auth não inicializado.");
                return;
            }
            try {
                await signOut(auth);
                // onAuthStateChanged will handle UI update
            } catch (err) {
                console.error("Erro durante o logout:", err);
                displayError(err.message);
            }
        }

        // Initialize Firebase and set up auth listener
        window.onload = async function() {
            try {
                const firebaseConfig = typeof __firebase_config !== 'undefined'
                    ? JSON.parse(__firebase_config)
                    : {};
                const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

                app = initializeApp(firebaseConfig);
                auth = getAuth(app);
                db = getFirestore(app);

                const initialAuthToken = typeof __initial_auth_token !== 'undefined'
                    ? __initial_auth_token
                    : null;

                if (initialAuthToken) {
                    await signInWithCustomToken(auth, initialAuthToken)
                        .catch((e) => {
                            console.error("Erro ao fazer login com token personalizado:", e);
                            signInAnonymously(auth); // Fallback to anonymous
                        });
                } else {
                    await signInAnonymously(auth);
                }

                // Set up authentication state listener
                onAuthStateChanged(auth, (user) => {
                    currentUser = user;
                    updateUI();
                });

                // Add event listeners
                authFormElement.addEventListener('submit', (e) => {
                    if (isRegistering) {
                        handleRegister(e);
                    } else {
                        handleLogin(e);
                    }
                });

                toggleAuthModeButton.addEventListener('click', () => {
                    isRegistering = !isRegistering;
                    updateUI();
                });

                logoutButton.addEventListener('click', handleLogout);

            } catch (e) {
                console.error("Falha ao inicializar o Firebase:", e);
                displayError("Falha ao inicializar o Firebase. Verifique o console para detalhes.");
                loadingScreen.classList.add('hidden');
                authFormDiv.classList.remove('hidden'); // Show form even on init error
            }
        };
    </script>
</body>
</html>
