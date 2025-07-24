<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Event Planner</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        /*
        *** IMPORTANT FIX FOR GITHUB PAGES MODAL ISSUE ***
        This rule ensures the modal is hidden immediately upon page load,
        addressing the "white box popup" issue before Tailwind CDN fully loads.
        */
        #eventDetailModal {
            display: none !important;
        }

        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6; /* Light gray background */
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            padding: 1rem;
            box-sizing: border-box;
        }

        /* Custom scrollbar for event list for a more refined look */
        #eventList::-webkit-scrollbar {
            width: 8px;
            height: 8px; /* For horizontal scrollbar, though not expected here */
        }
        #eventList::-webkit-scrollbar-track {
            background: #e0e7ff; /* Light indigo background for track */
            border-radius: 10px;
        }
        #eventList::-webkit-scrollbar-thumb {
            background: #818cf8; /* Indigo 400 for thumb */
            border-radius: 10px;
        }
        #eventList::-webkit-scrollbar-thumb:hover {
            background: #6366f1; /* Darker indigo on hover */
        }

        /* Modal overlay for consistent background */
        .modal-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.6); /* Slightly darker overlay */
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 1000;
            opacity: 0; /* Start hidden for smooth transition */
            visibility: hidden; /* Start hidden for smooth transition */
            transition: opacity 0.3s ease-out, visibility 0.3s ease-out;
        }

        .modal-overlay:not(.hidden) { /* When not hidden, make it visible and opaque */
            opacity: 1;
            visibility: visible;
        }

        .modal-content {
            background-color: white;
            padding: 2rem;
            border-radius: 0.75rem; /* rounded-xl */
            box-shadow: 0 15px 30px -5px rgba(0, 0, 0, 0.2), 0 6px 12px -2px rgba(0, 0, 0, 0.1); /* Stronger, more appealing shadow */
            max-width: 90%;
            width: 400px;
            position: relative;
            transform: translateY(20px); /* Start slightly below for slide-up effect */
            transition: transform 0.3s ease-out;
        }

        .modal-overlay:not(.hidden) .modal-content {
            transform: translateY(0); /* Slide up to final position */
        }

        /* Custom styles for focus, hover, and active states on buttons/inputs */
        .input-focus-style:focus {
            border-color: #6366f1; /* Indigo-500 for border */
            box-shadow: 0 0 0 3px rgba(99, 102, 241, 0.3); /* Ring shadow matching focus */
            outline: none;
        }

        .button-hover-focus-style {
            transition: background-color 0.2s ease, transform 0.1s ease, box-shadow 0.2s ease;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }
        .button-hover-focus-style:hover {
            transform: translateY(-2px); /* Slight lift on hover */
            box-shadow: 0 6px 10px rgba(0, 0, 0, 0.15);
        }
        .button-hover-focus-style:active {
            transform: translateY(0); /* Press down effect */
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.08);
        }
    </style>
</head>
<body class="bg-gray-100 flex items-center justify-center min-h-screen p-4">

    <div class="bg-white p-6 md:p-8 rounded-xl shadow-2xl w-full max-w-xl mx-auto border border-gray-100 transform transition-all duration-300 ease-in-out hover:shadow-3xl">
        <h1 class="text-4xl font-extrabold text-gray-900 mb-8 text-center tracking-tight">Event Planner</h1>

        <!-- Login Form / Logout Button Section -->
        <div class="mb-8 space-y-4">
            <div id="loginForm" class="space-y-4 p-5 border border-gray-200 rounded-xl shadow-lg bg-gray-50">
                <p class="text-center text-gray-700 font-semibold text-lg mb-3">Log In to Manage Events</p>
                <input type="email" id="loginEmail" placeholder="Email"
                       class="w-full px-4 py-2.5 border border-gray-300 rounded-lg shadow-sm input-focus-style sm:text-base transition duration-150 ease-in-out">
                <input type="password" id="loginPassword" placeholder="Password"
                       class="w-full px-4 py-2.5 border border-gray-300 rounded-lg shadow-sm input-focus-style sm:text-base transition duration-150 ease-in-out">
                <button id="loginBtn"
                        class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-4 rounded-lg button-hover-focus-style focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 text-lg">
                    Log In
                </button>
            </div>
            <button id="logoutBtn"
                    class="hidden w-full bg-red-600 hover:bg-red-700 text-white font-bold py-3 px-4 rounded-lg button-hover-focus-style focus:ring-2 focus:ring-red-500 focus:ring-offset-2 text-lg">
                Log Out
            </button>
            <p id="authStatusMessage" class="text-center text-base text-gray-600 mt-2 font-medium">Initializing...</p>
        </div>


        <div class="mb-6 space-y-4">
            <!-- Event Name Input with Quick-Select Buttons -->
            <div>
                <label for="eventName" class="block text-sm font-medium text-gray-700 mb-1">Event Name</label>
                <input type="text" id="eventName" placeholder="Enter event name"
                        class="mt-1 block w-full px-4 py-2.5 border border-gray-300 rounded-lg shadow-sm input-focus-style sm:text-base transition duration-150 ease-in-out">
                <!-- Quick-select buttons for default event names -->
                <div class="flex flex-wrap gap-2 mt-3">
                    <button id="btnDestinationWedding" type="button"
                            class="bg-blue-100 hover:bg-blue-200 text-blue-800 text-xs px-3.5 py-1.5 rounded-full font-semibold transition duration-150 ease-in-out button-hover-focus-style">
                        Destination Wedding
                    </button>
                    <button id="btnRoomsBooked" type="button"
                            class="bg-blue-100 hover:bg-blue-200 text-blue-800 text-xs px-3.5 py-1.5 rounded-full font-semibold transition duration-150 ease-in-out button-hover-focus-style">
                        Rooms Booked
                    </button>
                </div>
            </div>

            <!-- Event Description Textarea -->
            <div>
                <label for="eventDescription" class="block text-sm font-medium text-gray-700 mb-1">Description (Optional)</label>
                <textarea id="eventDescription" rows="3" placeholder="Add a brief description of the event"
                              class="mt-1 block w-full px-4 py-2.5 border border-gray-300 rounded-lg shadow-sm input-focus-style sm:text-base transition duration-150 ease-in-out resize-y"></textarea>
            </div>

            <!-- Starting Date Input -->
            <div>
                <label for="dateFrom" class="block text-sm font-medium text-gray-700 mb-1">Starting Date</label>
                <input type="date" id="dateFrom"
                        class="mt-1 block w-full px-4 py-2.5 border border-gray-300 rounded-lg shadow-sm input-focus-style sm:text-base transition duration-150 ease-in-out">
            </div>

            <!-- NEW: Number of Dates Input -->
            <div>
                <label for="numberOfDates" class="block text-sm font-medium text-gray-700 mb-1">Number of Dates</label>
                <input type="number" id="numberOfDates" placeholder="e.g., 3 for 3 days" min="1" value="1"
                        class="mt-1 block w-full px-4 py-2.5 border border-gray-300 rounded-lg shadow-sm input-focus-style sm:text-base transition duration-150 ease-in-out">
            </div>

            <!-- Add Event Button -->
            <button id="addEventBtn" disabled
                    class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 px-4 rounded-lg button-hover-focus-style focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed text-lg">
                Add Event
            </button>
        </div>

        <!-- Message Box for Validation/Errors -->
        <div id="messageBox" class="hidden bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded-lg relative mb-4 font-medium" role="alert">
            <span id="messageText" class="block sm:inline"></span>
            <span class="absolute top-0 bottom-0 right-0 px-4 py-3 cursor-pointer" onclick="document.getElementById('messageBox').classList.add('hidden');">
                <svg class="fill-current h-6 w-6 text-red-500" role="button" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20"><title>Close</title><path d="M14.348 14.849a1.2 1.2 0 0 1-1.697 0L10 11.103l-2.651 2.646a1.2 1.2 0 1 1-1.697-1.697L8.303 9.406l-2.651-2.646a1.2 1.2 0 1 1 1.697-1.697L10 7.71l2.651-2.646a1.2 1.2 0 0 1 1.697 1.697L11.697 9.406l2.651 2.646a1.2 1.2 0 0 1 0 1.697z"/></svg>
            </span>
        </div>

        <!-- Event List Display Area -->
        <div class="mt-8">
            <h2 class="text-3xl font-bold text-gray-800 mb-5 text-center">Upcoming Events</h2>

            <!-- Date Filter Section -->
            <div class="mb-6 flex flex-col sm:flex-row items-center space-y-2 sm:space-y-0 sm:space-x-2 p-3 bg-gray-50 rounded-lg border border-gray-100 shadow-inner">
                <label for="filterDateInput" class="text-sm font-medium text-gray-700 sm:flex-shrink-0">Filter by Date:</label>
                <input type="date" id="filterDateInput"
                        class="flex-grow px-3 py-2 border border-gray-300 rounded-md shadow-sm input-focus-style sm:text-sm">
                <button id="clearDateFilterBtn"
                        class="w-full sm:w-auto bg-gray-200 hover:bg-gray-300 text-gray-700 font-medium py-2 px-4 rounded-md text-sm transition duration-150 ease-in-out button-hover-focus-style">
                    Clear Filter
                </button>
            </div>

            <div id="eventList" class="space-y-4 max-h-80 overflow-y-auto pr-3 border border-gray-200 p-3 rounded-lg bg-white shadow-inner">
                <!-- Events will be added here -->
                <p id="noEventsMessage" class="text-gray-500 text-center italic py-4">No events added yet. Add your first event above!</p>
            </div>
        </div>
    </div>

    <!-- Event Detail Modal -->
    <div id="eventDetailModal" class="modal-overlay hidden">
        <div class="modal-content">
            <h2 id="modalEventName" class="text-2xl font-bold text-gray-800 mb-4 text-center"></h2>
            <div class="text-gray-700 mb-2">
                <span class="font-semibold">From:</span> <span id="modalDateFrom"></span>
            </div>
            <div class="text-gray-700 mb-4">
                <span class="font-semibold">To:</span> <span id="modalDateTo"></span>
            </div>
            <p id="modalEventDescription" class="text-gray-700 mb-6 whitespace-pre-wrap p-3 bg-gray-50 border border-gray-200 rounded-md"></p>
            <button id="closeModalBtn"
                    class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 px-4 rounded-lg button-hover-focus-style">
                Close
            </button>
        </div>
    </div>

    <script type="module">
        // --- Firebase SDK Imports (Version 12.0.0) ---
        import { initializeApp } from "https://www.gstatic.com/firebasejs/12.0.0/firebase-app.js";
        import { getAnalytics, logEvent } from "https://www.gstatic.com/firebasejs/12.0.0/firebase-analytics.js";
        // Updated imports for Email/Password authentication
        import { getAuth, signInWithEmailAndPassword, signOut, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/12.0.0/firebase-auth.js";
        import { getDatabase, ref, push, remove, onValue, query, orderByChild, serverTimestamp, update } from "https://www.gstatic.com/firebasejs/12.0.0/firebase-database.js";

        // --- Global Firebase Variables ---
        let app;
        let db;
        let auth;
        let analytics;
        let currentUserId = null;
        let allEvents = []; // To store all fetched events for client-side filtering
        let filterDate = null; // Stores the current date filter

        // --- Your Firebase Configuration ---
        // IMPORTANT: Ensure these are your exact Firebase project configuration values.
        const firebaseConfig = {
            apiKey: "AIzaSyBqjLWOahOwNE_lQ1ek9YddNztzdfJYMiM",
            authDomain: "my-busssiness-hrr.firebaseapp.com",
            databaseURL: "https://my-busssiness-hrr-default-rtdb.firebaseio.com",
            projectId: "my-busssiness-hrr",
            storageBucket: "my-busssiness-hrr.firebasestorage.app",
            messagingSenderId: "426018891071",
            appId: "1:426018891071:web:7ad54c177f30f446da1f8a",
            measurementId: "G-15CEW6N7N5"
        };

        const appId = firebaseConfig.appId;
        // This token is primarily for Canvas environment; for GitHub Pages, users will log in.
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        // Get references to HTML elements
        const eventNameInput = document.getElementById('eventName');
        const btnDestinationWedding = document.getElementById('btnDestinationWedding');
        const btnRoomsBooked = document.getElementById('btnRoomsBooked');
        const eventDescriptionInput = document.getElementById('eventDescription');
        const dateFromInput = document.getElementById('dateFrom');
        // REMOVED: const dateToInput = document.getElementById('dateTo');
        const numberOfDatesInput = document.getElementById('numberOfDates'); // NEW: Reference for number of dates input

        const addEventBtn = document.getElementById('addEventBtn');
        const eventList = document.getElementById('eventList');
        const noEventsMessage = document.getElementById('noEventsMessage');
        const messageBox = document.getElementById('messageBox');
        const messageText = document.getElementById('messageText');
        const authStatusMessage = document.getElementById('authStatusMessage');

        // New references for login/logout UI
        const loginEmailInput = document.getElementById('loginEmail');
        const loginPasswordInput = document.getElementById('loginPassword');
        const loginBtn = document.getElementById('loginBtn');
        const logoutBtn = document.getElementById('logoutBtn');
        const loginForm = document.getElementById('loginForm');

        // Modal elements
        const eventDetailModal = document.getElementById('eventDetailModal');
        const modalEventName = document.getElementById('modalEventName');
        const modalDateFrom = document.getElementById('modalDateFrom');
        const modalDateTo = document.getElementById('modalDateTo');
        const modalEventDescription = document.getElementById('modalEventDescription');
        const closeModalBtn = document.getElementById('closeModalBtn'); // Ref for close button

        // Date Filter elements
        const filterDateInput = document.getElementById('filterDateInput');
        const clearDateFilterBtn = document.getElementById('clearDateFilterBtn');

        // --- Quick-select buttons event listeners ---
        btnDestinationWedding.addEventListener('click', () => {
            eventNameInput.value = 'Destination Wedding';
        });

        btnRoomsBooked.addEventListener('click', () => {
            eventNameInput.value = 'Rooms Booked';
        });

        // --- Date Filter event listeners ---
        filterDateInput.addEventListener('input', () => {
            filterDate = filterDateInput.value;
            renderEvents(allEvents); // Re-render with the new filter
        });

        clearDateFilterBtn.addEventListener('click', () => {
            filterDateInput.value = '';
            filterDate = null;
            renderEvents(allEvents); // Re-render to show all events
        });

        // --- Modal close button event listener ---
        closeModalBtn.addEventListener('click', () => {
            eventDetailModal.classList.add('hidden'); // This is the line that hides the modal
        });


        // --- Utility Functions ---
        function showMessage(message, type = 'error') {
            messageText.textContent = message;
            messageBox.classList.remove('hidden');
            if (type === 'error') {
                messageBox.classList.remove('bg-green-100', 'border-green-400', 'text-green-700');
                messageBox.classList.add('bg-red-100', 'border-red-400', 'text-red-700');
            } else if (type === 'success') {
                messageBox.classList.remove('bg-red-100', 'border-red-400', 'text-red-700');
                messageBox.classList.add('bg-green-100', 'border-green-400', 'text-green-700');
            }
            // Hide message after 5 seconds, unless it's an error which might persist for debugging
            setTimeout(() => {
                hideMessage();
            }, 5000);
        }

        function hideMessage() {
            messageBox.classList.add('hidden');
        }

        function formatFriendlyDate(dateString) {
            if (!dateString) return '';
            const date = new Date(dateString + 'T00:00:00'); // Append T00:00:00 to ensure date is parsed as local day start
            const options = { year: 'numeric', month: 'long', day: 'numeric' };
            return date.toLocaleDateString(undefined, options);
        }

        // --- Realtime Database Listener Function ---
        function setupEventListener() {
            if (!currentUserId) {
                console.warn("User not authenticated yet, cannot set up Realtime Database listener.");
                // Clear events list if no user
                eventList.innerHTML = '<p class="text-gray-500 text-center italic py-4">Please log in to see your events.</p>';
                noEventsMessage.classList.add('hidden');
                return;
            }

            const eventsRef = ref(db, `artifacts/${appId}/users/${currentUserId}/events`);

            onValue(query(eventsRef, orderByChild('timestamp')), (snapshot) => {
                allEvents = []; // Reset the global array
                snapshot.forEach((childSnapshot) => {
                    allEvents.push({ id: childSnapshot.key, ...childSnapshot.val() });
                });
                renderEvents(allEvents); // Render all events, filtering will happen inside renderEvents
            }, (error) => {
                console.error("Error fetching events from Realtime Database: ", error);
                showMessage('Error loading events. Please refresh the page.', 'error');
            });
        }

        // --- Firebase Initialization and Authentication Function ---
        async function initializeFirebaseAndAuth() {
            if (Object.keys(firebaseConfig).length === 0) {
                console.error("Firebase config is missing or empty. Events will not be saved.");
                authStatusMessage.textContent = 'Firebase config missing!';
                addEventBtn.disabled = true;
                return;
            }

            app = initializeApp(firebaseConfig);
            db = getDatabase(app);
            auth = getAuth(app);
            analytics = getAnalytics(app);

            // Listen for auth state changes globally
            onAuthStateChanged(auth, async (user) => {
                if (user) {
                    currentUserId = user.uid;
                    addEventBtn.disabled = false;
                    authStatusMessage.textContent = `Logged in as: ${user.email || user.uid}`;
                    authStatusMessage.classList.remove('text-gray-600', 'text-red-700');
                    authStatusMessage.classList.add('text-green-700');

                    loginForm.classList.add('hidden'); // Hide login form
                    logoutBtn.classList.remove('hidden'); // Show logout button

                    setupEventListener(); // Set up database listener *after* user is authenticated
                } else {
                    currentUserId = null;
                    addEventBtn.disabled = true;
                    authStatusMessage.textContent = 'Please log in to manage events.';
                    authStatusMessage.classList.remove('text-green-700');
                    authStatusMessage.classList.add('text-red-700');

                    loginForm.classList.remove('hidden'); // Show login form
                    logoutBtn.classList.add('hidden'); // Hide logout button

                    eventList.innerHTML = '<p class="text-gray-500 text-center italic py-4">Please log in to see your events.</p>';
                    noEventsMessage.classList.add('hidden');
                }
            });

            // For GitHub Pages, we rely on the user to log in via the form.
            // The __initial_auth_token is primarily for the Canvas environment.
            if (initialAuthToken) {
                try {
                    await signInWithCustomToken(auth, initialAuthToken);
                } catch (e) {
                    console.error("Error signing in with custom token:", e);
                    // Fallback or just let onAuthStateChanged handle the "not logged in" state
                }
            }

            logEvent(analytics, 'app_started'); // Log app start regardless of auth method
        }

        // --- Login and Logout Functions ---
        loginBtn.addEventListener('click', async () => {
            hideMessage();
            const email = loginEmailInput.value;
            const password = loginPasswordInput.value;

            if (!email || !password) {
                showMessage('Please enter both email and password.', 'error');
                return;
            }

            try {
                await signInWithEmailAndPassword(auth, email, password);
                showMessage('Logged in successfully!', 'success');
                loginEmailInput.value = ''; // Clear fields on successful login
                loginPasswordInput.value = '';
            } catch (error) {
                console.error("Login failed:", error.code, error.message);
                let errorMessage = 'Login failed. Please check your email and password.';
                if (error.code === 'auth/user-not-found') {
                    errorMessage = 'No user found with this email.';
                } else if (error.code === 'auth/wrong-password') {
                    errorMessage = 'Incorrect password.';
                } else if (error.code === 'auth/invalid-credential') { // More generic for security
                    errorMessage = 'Invalid email or password.';
                }
                showMessage(errorMessage, 'error');
            }
        });

        logoutBtn.addEventListener('click', async () => {
            try {
                await signOut(auth);
                showMessage('Logged out successfully.', 'success');
            } catch (error) {
                console.error("Logout failed:", error.code, error.message);
                showMessage('Logout failed. Please try again.', 'error');
            }
        });


        // --- CRUD Operations for Realtime Database ---
        async function addEventToFirebase(eventName, eventDescription, dateFrom, dateTo) { // dateTo is now calculated
            if (!currentUserId) {
                showMessage('You must be logged in to add events.', 'error');
                return;
            }

            try {
                // Data stored under artifacts/{appId}/users/{userId}/events
                const userEventsRef = ref(db, `artifacts/${appId}/users/${currentUserId}/events`);

                await push(userEventsRef, {
                    eventName: eventName,
                    eventDescription: eventDescription,
                    dateFrom: dateFrom,
                    dateTo: dateTo, // Use the calculated 'dateTo'
                    isPaid: false, // Default to not paid
                    timestamp: serverTimestamp() // Adds a server-side timestamp for ordering
                });

                showMessage('Event added successfully!', 'success');
                eventNameInput.value = '';
                eventDescriptionInput.value = '';
                dateFromInput.value = '';
                numberOfDatesInput.value = '1'; // Reset number of dates to 1

            } catch (e) {
                console.error("Error adding event to Realtime Database: ", e);
                showMessage('Error adding event. Please try again.', 'error');
            }
        }

        async function toggleEventPaidStatus(eventId, currentIsPaidStatus) {
            if (!currentUserId) {
                showMessage('You must be logged in to update events.', 'error');
                return;
            }

            try {
                const eventRef = ref(db, `artifacts/${appId}/users/${currentUserId}/events/${eventId}`);
                await update(eventRef, {
                    isPaid: !currentIsPaidStatus // Toggle the status
                });
                showMessage(`Event marked as ${!currentIsPaidStatus ? 'Paid' : 'Unpaid'}.`, 'success');
            } catch (e) {
                console.error("Error updating paid status: ", e);
                showMessage('Error updating paid status. Please try again.', 'error');
            }
        }

        async function deleteEventFromFirebase(eventId) {
            if (!currentUserId) {
                showMessage('You must be logged in to delete events.', 'error');
                return;
            }

            try {
                const eventToDeleteRef = ref(db, `artifacts/${appId}/users/${currentUserId}/events/${eventId}`);
                await remove(eventToDeleteRef);
                showMessage('Event deleted successfully.', 'success');
            } catch (e) {
                console.error("Error deleting event from Realtime Database: ", e);
                showMessage('Error deleting event. Please try again.', 'error');
            }
        }

        // --- Function to Render Events (called by RTDB listener and filter) ---
        function renderEvents(events) {
            eventList.innerHTML = ''; // Clear existing events

            let eventsToRender = [...events]; // Create a copy of all events

            // Apply date filter if one is set
            if (filterDate) {
                eventsToRender = eventsToRender.filter(event => {
                    // Check if the filterDate falls within the event's date range
                    return event.dateFrom && event.dateTo && event.dateFrom <= filterDate && event.dateTo >= filterDate;
                });
            }

            if (eventsToRender.length === 0) {
                noEventsMessage.classList.remove('hidden');
                // Adjust message if a filter is active
                if (filterDate) {
                    noEventsMessage.textContent = `No events found for ${formatFriendlyDate(filterDate)}.`;
                } else {
                    noEventsMessage.textContent = 'No events added yet. Add your first event above!';
                }
            } else {
                noEventsMessage.classList.add('hidden'); // Hide if there are events
                eventsToRender.forEach(event => {
                    const eventItem = document.createElement('div');
                    // Enhanced styling for event items
                    eventItem.classList.add(
                        'bg-white', 'p-4', 'rounded-lg', 'shadow-md', 'border', 'border-gray-100',
                        'flex', 'flex-col', 'md:flex-row', 'justify-between', 'items-start', 'md:items-center', 'gap-3',
                        'transform', 'transition-all', 'duration-200', 'ease-in-out', 'hover:scale-[1.01]', 'hover:shadow-lg'
                    );

                    const eventDetails = document.createElement('div');
                    eventDetails.classList.add('mb-2', 'md:mb-0', 'cursor-pointer', 'flex-grow');
                    eventDetails.setAttribute('data-event-name', event.eventName || '');
                    eventDetails.setAttribute('data-date-from', event.dateFrom || '');
                    eventDetails.setAttribute('data-date-to', event.dateTo || '');
                    eventDetails.setAttribute('data-event-description', event.eventDescription || '');

                    const eventTitle = document.createElement('h3');
                    eventTitle.classList.add('text-lg', 'font-semibold', 'text-indigo-800');
                    eventTitle.textContent = event.eventName || 'No Name';

                    const eventDates = document.createElement('p');
                    eventDates.classList.add('text-sm', 'text-indigo-700');
                    eventDates.textContent = `${formatFriendlyDate(event.dateFrom)} to ${formatFriendlyDate(event.dateTo)}`;

                    eventDetails.appendChild(eventTitle);
                    eventDetails.appendChild(eventDates);

                    // Event listener for opening the detail modal
                    eventDetails.addEventListener('click', () => {
                        modalEventName.textContent = event.eventName || 'No Name';
                        modalDateFrom.textContent = formatFriendlyDate(event.dateFrom);
                        modalDateTo.textContent = formatFriendlyDate(event.dateTo);
                        // Ensure description is shown, with a fallback message
                        modalEventDescription.textContent = event.eventDescription ? `Description: ${event.eventDescription}` : 'No description provided for this event.';
                        eventDetailModal.classList.remove('hidden'); // Show the modal
                        hideMessage(); // Hide any general messages when modal opens
                    });

                    // --- Paid Button ---
                    const paidButton = document.createElement('button');
                    // Enhanced styling for paid button
                    paidButton.classList.add('font-medium', 'py-1.5', 'px-3.5', 'rounded-full', 'text-xs', 'transition', 'duration-150', 'ease-in-out', 'flex-shrink-0', 'button-hover-focus-style');
                    if (event.isPaid) {
                        paidButton.textContent = 'Paid';
                        paidButton.classList.add('bg-green-500', 'hover:bg-green-600', 'text-white');
                        paidButton.classList.remove('bg-yellow-500');
                    } else {
                        paidButton.textContent = 'Mark Paid';
                        paidButton.classList.add('bg-yellow-500', 'hover:bg-yellow-600', 'text-white');
                        paidButton.classList.remove('bg-green-500');
                    }
                    paidButton.addEventListener('click', () => {
                        toggleEventPaidStatus(event.id, event.isPaid);
                    });

                    // --- Delete Button ---
                    const deleteButton = document.createElement('button');
                    // Enhanced styling for delete button
                    deleteButton.classList.add('bg-red-500', 'hover:bg-red-600', 'text-white', 'font-medium', 'py-1.5', 'px-3.5', 'rounded-full', 'text-xs', 'transition', 'duration-150', 'ease-in-out', 'flex-shrink-0', 'button-hover-focus-style');
                    deleteButton.textContent = 'Delete';
                    deleteButton.addEventListener('click', () => {
                        deleteEventFromFirebase(event.id);
                    });

                    // Group buttons in a container for better layout and responsiveness
                    const buttonContainer = document.createElement('div');
                    buttonContainer.classList.add('flex', 'flex-col', 'sm:flex-row', 'gap-2', 'items-center', 'justify-end', 'sm:w-auto', 'w-full');
                    buttonContainer.appendChild(paidButton);
                    buttonContainer.appendChild(deleteButton);

                    eventItem.appendChild(eventDetails);
                    eventItem.appendChild(buttonContainer);

                    eventList.appendChild(eventItem);
                });
            }
        }

        // --- Main Event Listeners (Add Event) ---
        addEventBtn.addEventListener('click', () => {
            hideMessage();

            const eventName = eventNameInput.value.trim();
            const eventDescription = eventDescriptionInput.value.trim();
            const startingDate = dateFromInput.value; // Renamed for clarity
            const numberOfDates = parseInt(numberOfDatesInput.value); // Get as integer

            if (!eventName || !startingDate) {
                showMessage('Please fill in Event Name and Starting Date.', 'error');
                return;
            }

            if (isNaN(numberOfDates) || numberOfDates < 1) {
                showMessage('Number of dates must be a positive number (e.g., 1 or more).', 'error');
                return;
            }

            // Calculate the end date
            const startDateObj = new Date(startingDate + 'T00:00:00'); // Ensure consistent date object
            const endDateObj = new Date(startDateObj); // Start with the same date
            endDateObj.setDate(startDateObj.getDate() + numberOfDates - 1); // Add number of days - 1 (since start date counts as 1st day)

            // Format endDateObj back to YYYY-MM-DD for consistency and Firebase storage
            const calculatedDateTo = endDateObj.toISOString().split('T')[0];

            addEventToFirebase(eventName, eventDescription, startingDate, calculatedDateTo); // Pass calculated dateTo
        });
        
        // --- Initial Call to Start Firebase ---
        initializeFirebaseAndAuth();
    </script>
</body>
</html>
