<script lang="ts">
    import Header from './Header.svelte';

    // Login form state
    let email = $state("ntap1@spatialdesignhub.com");
    let password = $state("ntap2025");
    let loading = $state(false);
    let response = $state("");
    let error = $state("");

    // JWT and spatial models state
    let jwtToken = $state("");
    let modelsLoading = $state(false);
    let modelsData = $state("");
    let modelsError = $state("");

    // Original demo state
    let name = $state("kasra")
    let status : 'OPEN' | 'CLOSED' = $state('OPEN')
    let derivedRes = $derived(name + " " + "niroumand")

        async function handleLogin(event: Event) {
        event.preventDefault();
        loading = true;
        error = "";
        response = "";

        try {
            const res = await fetch('https://3w6dargl5c.execute-api.eu-west-2.amazonaws.com/dev/login', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    email: email,
                    password: password
                })
            });

            const data = await res.json();

            if (res.ok) {
                response = `Login successful! Response: ${JSON.stringify(data, null, 2)}`;
                // Extract JWT token from response
                if (data.token) {
                    jwtToken = data.token;
                } else if (data.accessToken) {
                    jwtToken = data.accessToken;
                } else if (data.jwt) {
                    jwtToken = data.jwt;
                } else {
                    // Try to find token in nested objects
                    const tokenKey = Object.keys(data).find(key =>
                        key.toLowerCase().includes('token') || key.toLowerCase().includes('jwt')
                    );
                    if (tokenKey) {
                        jwtToken = data[tokenKey];
                    }
                }
            } else {
                error = `Login failed: ${data.message || 'Unknown error'}`;
            }
        } catch (err) {
            error = `Network error: ${err instanceof Error ? err.message : 'Unknown error'}`;
        } finally {
            loading = false;
        }
    }

    async function fetchSpatialModels() {
        if (!jwtToken) {
            modelsError = "Please login first to get a JWT token";
            return;
        }

        modelsLoading = true;
        modelsError = "";
        modelsData = "";

        try {
            const res = await fetch('https://3w6dargl5c.execute-api.eu-west-2.amazonaws.com/dev/spatial/models', {
                method: 'GET',
                headers: {
                    'Authorization': `Bearer ${jwtToken}`,
                    'Content-Type': 'application/json',
                }
            });

            const data = await res.json();

            if (res.ok) {
                modelsData = `Spatial models fetched successfully! Response: ${JSON.stringify(data, null, 2)}`;
            } else {
                modelsError = `Failed to fetch spatial models: ${data.message || 'Unknown error'}`;
            }
        } catch (err) {
            modelsError = `Network error: ${err instanceof Error ? err.message : 'Unknown error'}`;
        } finally {
            modelsLoading = false;
        }
    }

    function onclick() {
        status = status === "OPEN" ? "CLOSED" : "OPEN"
    }
</script>

<Header
name={name}
full_name={derivedRes}
/>

<!-- Login Form -->
<div class="login-container">
    <h2>Login</h2>
    <form onsubmit={handleLogin}>
        <div class="form-group">
            <label for="email">Email:</label>
            <input
                type="email"
                id="email"
                bind:value={email}
                placeholder="Enter your email"
                required
                disabled={loading}
            />
        </div>

        <div class="form-group">
            <label for="password">Password:</label>
            <input
                type="password"
                id="password"
                bind:value={password}
                placeholder="Enter your password"
                required
                disabled={loading}
            />
        </div>

        <button type="submit" disabled={loading || !email || !password}>
            {loading ? 'Logging in...' : 'Login'}
        </button>
    </form>

    {#if response}
        <div class="success">
            <pre>{response}</pre>
        </div>
    {/if}

    {#if error}
        <div class="error">
            {error}
        </div>
    {/if}

    {#if jwtToken}
        <div class="jwt-info">
            <strong>JWT Token Available:</strong> {jwtToken.substring(0, 50)}...
        </div>
    {/if}
</div>

<!-- Spatial Models Section -->
{#if jwtToken}
<div class="models-container">
    <h2>Spatial Models</h2>
    <p>Use your JWT token to fetch spatial models from the API</p>

    <button
        onclick={fetchSpatialModels}
        disabled={modelsLoading}
        class="models-button"
    >
        {modelsLoading ? 'Fetching Models...' : 'Fetch Spatial Models'}
    </button>

    {#if modelsData}
        <div class="success">
            <pre>{modelsData}</pre>
        </div>
    {/if}

    {#if modelsError}
        <div class="error">
            {modelsError}
        </div>
    {/if}
</div>
{/if}

<!-- Original Demo Content -->
<div class="demo-container">
    <h2>Demo Content</h2>
    <input type="text" bind:value={name} />
    <button onclick={() => {
        status = status === "OPEN" ? "CLOSED" : "OPEN"
    }}>get status</button>
    <p>{derivedRes}</p>
</div>

<style>
    .login-container {
        max-width: 400px;
        margin: 2rem auto;
        padding: 2rem;
        border: 1px solid #ddd;
        border-radius: 8px;
        background-color: #f9f9f9;
    }

    .login-container h2 {
        text-align: center;
        margin-bottom: 1.5rem;
        color: #333;
    }

    .form-group {
        margin-bottom: 1rem;
    }

    .form-group label {
        display: block;
        margin-bottom: 0.5rem;
        font-weight: bold;
        color: #555;
    }

    .form-group input {
        width: 100%;
        padding: 0.75rem;
        border: 1px solid #ccc;
        border-radius: 4px;
        font-size: 1rem;
        box-sizing: border-box;
    }

    .form-group input:focus {
        outline: none;
        border-color: #007bff;
        box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.25);
    }

    .form-group input:disabled {
        background-color: #f5f5f5;
        cursor: not-allowed;
    }

    button[type="submit"] {
        width: 100%;
        padding: 0.75rem;
        background-color: #007bff;
        color: white;
        border: none;
        border-radius: 4px;
        font-size: 1rem;
        cursor: pointer;
        transition: background-color 0.2s;
    }

    button[type="submit"]:hover:not(:disabled) {
        background-color: #0056b3;
    }

    button[type="submit"]:disabled {
        background-color: #6c757d;
        cursor: not-allowed;
    }

    .success {
        margin-top: 1rem;
        padding: 1rem;
        background-color: #d4edda;
        border: 1px solid #c3e6cb;
        border-radius: 4px;
        color: #155724;
    }

    .success pre {
        margin: 0;
        white-space: pre-wrap;
        word-break: break-word;
    }

    .error {
        margin-top: 1rem;
        padding: 1rem;
        background-color: #f8d7da;
        border: 1px solid #f5c6cb;
        border-radius: 4px;
        color: #721c24;
    }

    .demo-container {
        max-width: 400px;
        margin: 2rem auto;
        padding: 2rem;
        border: 1px solid #ddd;
        border-radius: 8px;
        background-color: #f0f8ff;
    }

    .demo-container h2 {
        text-align: center;
        margin-bottom: 1.5rem;
        color: #333;
    }

    .demo-container input {
        width: 100%;
        padding: 0.5rem;
        margin-bottom: 1rem;
        border: 1px solid #ccc;
        border-radius: 4px;
        box-sizing: border-box;
    }

    .demo-container button {
        padding: 0.5rem 1rem;
        background-color: #28a745;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
        margin-bottom: 1rem;
    }

    .demo-container button:hover {
        background-color: #218838;
    }

    .jwt-info {
        margin-top: 1rem;
        padding: 0.75rem;
        background-color: #e7f3ff;
        border: 1px solid #b8daff;
        border-radius: 4px;
        color: #004085;
        font-family: monospace;
        font-size: 0.875rem;
        word-break: break-all;
    }

    .models-container {
        max-width: 600px;
        margin: 2rem auto;
        padding: 2rem;
        border: 1px solid #ddd;
        border-radius: 8px;
        background-color: #f8f9fa;
    }

    .models-container h2 {
        text-align: center;
        margin-bottom: 1rem;
        color: #333;
    }

    .models-container p {
        text-align: center;
        color: #666;
        margin-bottom: 1.5rem;
    }

    .models-button {
        display: block;
        width: 100%;
        padding: 0.75rem;
        background-color: #17a2b8;
        color: white;
        border: none;
        border-radius: 4px;
        font-size: 1rem;
        cursor: pointer;
        transition: background-color 0.2s;
        margin-bottom: 1rem;
    }

    .models-button:hover:not(:disabled) {
        background-color: #138496;
    }

    .models-button:disabled {
        background-color: #6c757d;
        cursor: not-allowed;
    }
</style>
