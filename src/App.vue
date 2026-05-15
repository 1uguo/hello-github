<script setup lang="ts">
</script>

<template>
  <main class="hero">
    <h1 class="hero__heading">Hello Github</h1>
    <hr class="hero__rule" />
    <p class="hero__tagline">Open source begins with a single commit.</p>
    <footer class="hero__footer">
      <a
        href="https://github.com"
        target="_blank"
        rel="noopener noreferrer"
      >
        github.com
      </a>
    </footer>
  </main>
</template>

<style scoped>
.hero {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  padding: 2rem;
  position: relative;
}

.hero__heading {
  font-size: clamp(3rem, 8vw, 7rem);
  font-weight: 300;
  letter-spacing: -0.03em;
  line-height: 1.05;
  color: #1a1a1a;
  text-align: center;
  animation: fade-up 0.8s ease both;
}

.hero__rule {
  width: 120px;
  height: 1px;
  border: none;
  background-color: #e0ded8;
  margin: 3rem 0;
  animation: fade-up 0.8s 0.2s ease both;
}

.hero__tagline {
  font-size: 1.125rem;
  font-weight: 400;
  letter-spacing: 0.08em;
  color: #8a8a82;
  text-align: center;
  text-transform: uppercase;
  animation: fade-up 0.8s 0.35s ease both;
}

.hero__footer {
  position: absolute;
  bottom: 2rem;
  font-size: 0.875rem;
  letter-spacing: 0.04em;
  animation: fade-up 0.8s 0.5s ease both;
}

@keyframes fade-up {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
