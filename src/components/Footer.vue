<script>
export default {
    data() {
        return {
            currentYear: new Date().getFullYear()
        };
    }
}
</script>

<template>
  <footer class="footer">
    <p>© {{ currentYear }} MastermindAppMobile. All rights reserved.</p>
  </footer>
</template>

<style scoped>
.footer {
  width: 100%;
  color: #333;
  margin-top: auto;
  text-align: center;
}

.footer p {
  margin: 0;
  font-size: 0.9rem;
}
</style>