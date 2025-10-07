<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>One-Time Password (OTP) Flow</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@100..900&display=swap');
        body { font-family: 'Inter', sans-serif; background-color: #f7f9fb; }
        .card { max-width: 400px; }
        /* Hides the up/down arrows on number inputs */
        input[type="number"]::-webkit-outer-spin-button,
        input[type="number"]::-webkit-inner-spin-button {
            -webkit-appearance: none;
            margin: 0;
        }
        input[type="number"] {
            -moz-appearance: textfield;
        }
    </style>
</head>
<body>

<div id="app" class="min-h-screen flex items-center justify-center p-4">
    <!-- Content will be rendered here by JavaScript -->
</div>

<script>
    // --- Global State Management (Simulating React/Vue state) ---
    let state = {
        phoneNumber: '',
        otpCode: '',
        stage: 'input_phone', // 'input_phone', 'verify_otp', 'success', 'error'
        message: 'Enter your phone number to proceed.',
        isSubmitting: false,
    };

    /**
     * Updates the global state and triggers a re-render of the application.
     * @param {object} newState - The partial state object to merge.
     */
    function setState(newState) {
        state = { ...state, ...newState };
        // This call to render() is why we must ensure no infinite loops occur inside rendering functions.
        render();
    }

    // --- Utility Functions ---

    /**
     * Exponential backoff utility for retrying API calls.
     * @param {Function} func - The async function to execute.
     * @param {number} maxRetries - Maximum number of retries.
     * @returns {Promise<any>} The result of the function.
     */
    async function retryWithBackoff(func, maxRetries = 5) {
        for (let attempt = 0; attempt < maxRetries; attempt++) {
            try {
                return await func();
            } catch (error) {
                if (attempt === maxRetries - 1) {
                    throw error;
                }
                const delay = Math.pow(2, attempt) * 1000 + Math.random() * 1000;
                await new Promise(resolve => setTimeout(resolve, delay));
            }
        }
    }

    // --- Authentication Logic (Simulated) ---

    /**
     * Simulates sending an OTP code to the provided phone number.
     */
    async function mockSendOtp() {
        if (state.isSubmitting) return;

        const phone = state.phoneNumber.replace(/[^0-9]/g, '');
        if (phone.length !== 10) {
            setState({ message: 'Please enter a valid 10-digit phone number.', stage: 'input_phone', isSubmitting: false });
            return;
        }

        // Set isSubmitting and trigger a render to show the loading state
        setState({ isSubmitting: true, message: 'Sending OTP...', stage: 'input_phone' });

        try {
            await retryWithBackoff(async () => {
                // Simulate an API call delay
                await new Promise(resolve => setTimeout(resolve, Math.random() * 1000 + 500));

                // Success condition: Move to OTP verification stage
                setState({
                    stage: 'verify_otp',
                    message: `OTP sent to +1 (${phone.substring(0, 3)}) ${phone.substring(3, 6)}-${phone.substring(6)}. Use 123456 to test.`,
                    isSubmitting: false,
                    otpCode: ''
                });
            });

        } catch (error) {
            console.error("Error sending OTP:", error);
            setState({
                stage: 'error',
                message: 'Failed to send OTP. Please try again later.',
                isSubmitting: false
            });
        }
    }

    /**
     * Simulates verifying the provided OTP code.
     */
    async function mockVerifyOtp() {
        if (state.isSubmitting) return;

        const otp = state.otpCode.replace(/[^0-9]/g, '');
        if (otp.length !== 6) {
            setState({ message: 'Please enter the 6-digit code.', stage: 'verify_otp', isSubmitting: false });
            return;
        }

        setState({ isSubmitting: true, message: 'Verifying code...' });

        try {
            await retryWithBackoff(async () => {
                // Simulate an API call delay
                await new Promise(resolve => setTimeout(resolve, Math.random() * 1000 + 500));

                // Mock verification: '123456' is the success code
                if (otp === '123456') {
                    setState({
                        stage: 'success',
                        message: 'Verification successful! You are now logged in.',
                        isSubmitting: false
                    });
                } else {
                    throw new Error('Invalid OTP code.');
                }
            });

        } catch (error) {
            console.error("Error verifying OTP:", error);
            setState({
                stage: 'verify_otp',
                message: 'The code is incorrect. Please try again.',
                isSubmitting: false
            });
        }
    }

    // --- Event Handlers (Called by HTML onclick) ---

    /**
     * Handles input for the phone number field, cleaning and enforcing the 10-digit limit.
     */
    function handlePhoneInput(event) {
        let value = event.target.value.replace(/[^0-9]/g, '');
        // Enforce max 10 digits as required by validation
        value = value.substring(0, 10);
        event.target.value = value; // Update the displayed value

        setState({ phoneNumber: value });
        
        // Reset stage if they were previously in a final error/success state
        if (state.stage !== 'input_phone' && state.stage !== 'verify_otp') {
            setState({ stage: 'input_phone' });
        }
    }

    /**
     * Handles input for the OTP code field.
     */
    function handleOtpInput(event) {
        // Enforce max 6 digits
        let value = event.target.value.replace(/[^0-9]/g, '');
        value = value.substring(0, 6);
        event.target.value = value; // Update the displayed value

        setState({ otpCode: value });
    }

    function handleSendOtpClick() {
        mockSendOtp();
    }

    function handleVerifyOtpClick() {
        mockVerifyOtp();
    }

    function handleGoBack() {
        setState({
            stage: 'input_phone',
            message: 'Enter your phone number to proceed.',
            otpCode: '',
            isSubmitting: false
        });
    }

    // --- Render Functions ---

    /**
     * Renders the Phone Number Input stage.
     * @returns {string} HTML string.
     */
    function renderInputPhone() {
        const disabledAttr = state.isSubmitting ? 'disabled' : '';
        const btnClass = state.isSubmitting ? 'bg-gray-400 cursor-not-allowed' : 'bg-blue-600 hover:bg-blue-700';

        return `
            <input
                type="number"
                id="phone-input"
                class="w-full p-3 mb-4 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 transition duration-150 ease-in-out"
                placeholder="e.g., 5551234567 (10 digits)"
                value="${state.phoneNumber}"
                oninput="handlePhoneInput(event)"
                maxlength="10"
                ${disabledAttr}
            />
            <button
                onclick="handleSendOtpClick()"
                class="w-full text-white font-semibold py-3 px-4 rounded-lg shadow-md transition duration-300 ease-in-out ${btnClass}"
                ${disabledAttr}
            >
                ${state.isSubmitting ? 'Sending...' : 'Send OTP'}
            </button>
            <div class="text-sm text-center mt-4 text-gray-500">
                A 6-digit verification code will be sent to this number.
            </div>
        `;
    }

    /**
     * Renders the OTP Verification stage.
     * @returns {string} HTML string.
     */
    function renderOtpVerification() {
        const disabledAttr = state.isSubmitting ? 'disabled' : '';
        const btnClass = state.isSubmitting ? 'bg-gray-400 cursor-not-allowed' : 'bg-green-600 hover:bg-green-700';

        return `
            <div class="flex flex-col items-center">
                <input
                    type="number"
                    id="otp-input"
                    class="text-center tracking-widest text-xl w-full p-4 mb-4 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-green-500 transition duration-150 ease-in-out"
                    placeholder="Enter 6-digit code"
                    maxlength="6"
                    value="${state.otpCode}"
                    oninput="handleOtpInput(event)"
                    ${disabledAttr}
                />
                <button
                    onclick="handleVerifyOtpClick()"
                    class="w-full text-white font-semibold py-3 px-4 rounded-lg shadow-md transition duration-300 ease-in-out ${btnClass} mb-3"
                    ${disabledAttr}
                >
                    ${state.isSubmitting ? 'Verifying...' : 'Verify Code'}
                </button>
                <div class="flex justify-between w-full text-sm mt-2">
                    <button onclick="handleGoBack()" class="text-blue-500 hover:text-blue-700 font-medium transition duration-150 ease-in-out" ${disabledAttr}>
                        &larr; Change Number
                    </button>
                    <button onclick="handleSendOtpClick()" class="text-blue-500 hover:text-blue-700 font-medium transition duration-150 ease-in-out" ${disabledAttr}>
                        Resend Code
                    </button>
                </div>
            </div>
        `;
    }

    /**
     * Renders the Success or Error screens.
     * @returns {string} HTML string.
     */
    function renderFinalScreen() {
        const isSuccess = state.stage === 'success';
        const icon = isSuccess ?
            `<svg class="w-12 h-12 text-green-500" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg>` :
            `<svg class="w-12 h-12 text-red-500" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg>`;

        const title = isSuccess ? 'Success!' : 'Oops!';
        const btnClass = isSuccess ? 'bg-green-600 hover:bg-green-700' : 'bg-red-600 hover:bg-red-700';
        const buttonText = isSuccess ? 'Continue to App' : 'Try Again';

        return `
            <div class="flex flex-col items-center p-6 text-center">
                ${icon}
                <h2 class="text-2xl font-bold mt-4 mb-2 text-gray-800">${title}</h2>
                <p class="text-gray-600 mb-6">${state.message}</p>
                <button
                    onclick="handleGoBack()"
                    class="w-full text-white font-semibold py-3 px-4 rounded-lg shadow-md transition duration-300 ease-in-out ${btnClass}"
                >
                    ${buttonText}
                </button>
            </div>
        `;
    }

    /**
     * Main render function that updates the DOM based on current state.
     */
    function render() {
        const app = document.getElementById('app');
        let content = '';
        let title = '';

        switch (state.stage) {
            case 'input_phone':
                content = renderInputPhone();
                title = 'Phone Verification';
                break;
            case 'verify_otp':
                content = renderOtpVerification();
                title = 'Verify OTP Code';
                break;
            case 'success':
            case 'error':
                content = renderFinalScreen();
                title = state.stage === 'success' ? 'Verified' : 'Error';
                break;
            default:
                content = '<div>An unknown error occurred.</div>';
                title = 'Error';
        }

        // The main card structure
        app.innerHTML = `
            <div class="card bg-white p-6 md:p-8 rounded-xl shadow-2xl w-full">
                <h1 class="text-3xl font-extrabold text-gray-900 mb-6 text-center">${title}</h1>
                <p class="text-center text-sm text-gray-500 mb-6 h-10">${state.message}</p>
                ${content}
            </div>
        `;

        // Re-attach event listeners after innerHTML update (vanilla JS requirement)
        const phoneInput = document.getElementById('phone-input');
        if (phoneInput) {
            phoneInput.oninput = handlePhoneInput;
        }
        const otpInput = document.getElementById('otp-input');
        if (otpInput) {
            otpInput.oninput = handleOtpInput;
        }
    }

    // Initialize the application
    window.onload = () => {
        // Expose handlers globally so the inline HTML onclick attributes can access them
        window.handlePhoneInput = handlePhoneInput;
        window.handleOtpInput = handleOtpInput;
        window.handleSendOtpClick = handleSendOtpClick;
        window.handleVerifyOtpClick = handleVerifyOtpClick;
        window.handleGoBack = handleGoBack;

        // Initial render
        render();
    };
</script>

</body>
</html>
