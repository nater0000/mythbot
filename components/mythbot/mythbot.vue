<template>
  
  Hello World
</template>

<script setup>

</script>

<style>
/* Paste the entire <style> block from mythbot.html here */
:root {
    --bg-main: #1D4241;
    --bg-card: #183332;
    --text-main: #E0E0E0;
    --text-heading: #D4AF37; /* Gold */
    --text-muted: #A0A0A0;
    --border-color: #2c5d5b;
    --accent-turqoise: #5FE2DF;
    --accent-turqoise-dark: #4ac0bd;
    --accent-red: #a43743; /* Muted Crimson for text */
}
body {
    font-family: 'Lora', serif;
    background-color: var(--bg-main);
    color: var(--text-main);
}
h1, h2, h3, h4 {
    font-family: 'Cinzel', serif;
    color: var(--text-heading);
    letter-spacing: 0.05em;
}
.content-card {
    background-color: var(--bg-card);
    border: 1px solid var(--border-color);
    border-radius: 0.5rem;
    padding: 2.5rem;
    box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.2), 0 4px 6px -2px rgba(0, 0, 0, 0.1);
}
.nav-link {
    transition: all 0.2s ease-in-out;
    border-left: 3px solid transparent;
    color: var(--text-muted);
    padding: 0.75rem 1rem;
}
.nav-link.active {
    background-color: rgba(95, 226, 223, 0.05);
    color: var(--accent-turqoise);
    border-left-color: var(--accent-turqoise);
}
.nav-link:hover {
    background-color: rgba(95, 226, 223, 0.1);
    color: #fff;
}
textarea, .compass-input {
    background-color: rgba(0,0,0,0.2);
    border: 1px solid var(--border-color);
    border-radius: 0.5rem;
    padding: 0.75rem;
    width: 100%;
    transition: border-color 0.2s ease-in-out;
    font-size: 1rem;
    color: var(--text-main);
    font-family: 'Lora', serif;
}
textarea:focus, .compass-input:focus {
    outline: none;
    border-color: var(--accent-turqoise);
    box-shadow: 0 0 0 2px rgba(95, 226, 223, 0.3);
}
.btn-primary {
    background-color: var(--accent-turqoise);
    color: #0c2a29;
    padding: 0.75rem 1.5rem;
    border-radius: 0.5rem;
    font-family: 'Cinzel', serif;
    font-weight: 700;
    transition: all 0.2s ease-in-out;
    cursor: pointer;
    text-transform: uppercase;
    letter-spacing: 0.1em;
}
.btn-primary:hover {
    background-color: var(--accent-turqoise-dark);
    box-shadow: 0 0 15px rgba(95, 226, 223, 0.4);
}
.btn-secondary {
    background-color: transparent;
    color: var(--accent-turqoise);
    border: 1px solid var(--accent-turqoise);
    padding: 0.5rem 1rem;
    border-radius: 0.5rem;
    font-family: 'Cinzel', serif;
    font-weight: 700;
    transition: all 0.2s ease-in-out;
    cursor: pointer;
}
.btn-secondary:hover {
    background-color: rgba(95, 226, 223, 0.1);
}
.artifiction-icon {
    font-size: 2.5rem;
    margin-right: 1.5rem;
    line-height: 1;
}
.stage-content { display: none; }
.stage-content.active {
    display: block;
    animation: fadeIn 0.6s ease-in-out;
}
@keyframes fadeIn {
    from { opacity: 0; transform: translateY(15px); }
    to { opacity: 1; transform: translateY(0); }
}
.chart-container {
    position: relative;
    width: 100%;
    max-width: 500px;
    height: auto;
    margin: 2rem auto;
}
.prose {
    color: var(--text-main);
    line-height: 1.8;
}
.prose h3 {
    font-size: 1.5rem;
    font-weight: 700;
    margin-top: 2em;
    margin-bottom: 0.5em;
    color: var(--text-heading);
    border-bottom: 1px solid var(--border-color);
    padding-bottom: 0.5rem;
}
.prose p { margin-bottom: 1.25em; }
.prose strong { color: var(--accent-turqoise); }
.nav-artifact-header {
    font-family: 'Cinzel', serif;
    color: #fff;
    font-weight: 700;
    letter-spacing: 0.1em;
}
.crimson-text { color: var(--accent-red); }
.synthesis-box {
    background-color: rgba(0,0,0,0.1);
    border-left: 4px solid var(--accent-turqoise);
    padding: 1.5rem;
    border-radius: 0.5rem;
    font-family: 'Lora', serif;
    font-style: italic;
    color: #E5E7EB;
}
.ornamental-divider {
    height: 1px;
    background: linear-gradient(to right, transparent, var(--border-color), transparent);
    margin: 3rem 0;
}
input[type=range] {
    -webkit-appearance: none;
    width: 100%;
    background: transparent;
}
input[type=range]:focus {
    outline: none;
}
input[type=range]::-webkit-slider-runnable-track {
    width: 100%;
    height: 4px;
    cursor: pointer;
    background: var(--border-color);
    border-radius: 5px;
}
input[type=range]::-webkit-slider-thumb {
    -webkit-appearance: none;
    border: 2px solid var(--bg-main);
    height: 20px;
    width: 20px;
    border-radius: 50%;
    background: var(--accent-turqoise);
    cursor: pointer;
    margin-top: -8px;
}
input[type=range]::-moz-range-track {
    width: 100%;
    height: 4px;
    cursor: pointer;
    background: var(--border-color);
    border-radius: 5px;
}
input[type=range]::-moz-range-thumb {
    border: 2px solid var(--bg-main);
    height: 20px;
    width: 20px;
    border-radius: 50%;
    background: var(--accent-turqoise);
    cursor: pointer;
}
</style>