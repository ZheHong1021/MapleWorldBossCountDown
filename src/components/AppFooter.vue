<template>
  <footer class="app-footer">
    <div class="footer-content">
      <p class="data-source">
        📊 資料來源：
        <a 
          href="https://docs.google.com/spreadsheets/d/1xb6FhSGcM6EIj7aDcUmxZA9zVxLEw8xxXxxej5Tcrt4/htmlview?gid=2100610320#gid=2100610320" 
          target="_blank" 
          rel="noopener noreferrer"
          class="source-link"
        >
          中英文 Artale Monster Database
        </a>
      </p>
      <p class="copyright">
        Made with ❤️ for MapleStory players
      </p>
    </div>
  </footer>
</template>

<script>
export default {
  name: 'AppFooter'
}
</script>

<style scoped>
.app-footer {
  width: calc(100% + 4rem);
  margin-left: -2rem;
  margin-right: -2rem;
  margin-top: 3rem;
  padding: 2rem 2rem 1rem 2rem;
  border-top: 1px solid #333;
  background-color: rgba(0, 0, 0, 0.1);
  text-align: center;
  transition: all 0.3s ease;
  box-sizing: border-box;
}

.footer-content {
  max-width: 600px;
  margin: 0 auto;
}

.data-source {
  font-size: 0.95rem;
  margin-bottom: 0.5rem;
  color: #888;
  line-height: 1.5;
}

.source-link {
  color: #61dafb;
  text-decoration: none;
  font-weight: 600;
  padding: 0.2rem 0.5rem;
  border-radius: 4px;
  transition: all 0.3s ease;
  display: inline-block;
}

.source-link:hover {
  color: #ffffff;
  background-color: #61dafb;
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(97, 218, 251, 0.3);
}

.copyright {
  font-size: 0.85rem;
  color: #666;
  margin: 0;
  font-style: italic;
}

/* 響應式設計 */
@media (max-width: 768px) {
  .app-footer {
    width: calc(100% + 4rem);
    margin-left: -2rem;
    margin-right: -2rem;
    padding: 1.5rem 1rem 1rem 1rem;
  }

  .data-source {
    font-size: 0.9rem;
  }

  .copyright {
    font-size: 0.8rem;
  }

  .source-link {
    padding: 0.3rem 0.6rem;
    font-size: 0.9rem;
  }
}
</style>

<!-- 主題相關的全域樣式 -->
<style>
/* 深色主題下的 footer */
body.dark-theme .app-footer {
  border-top-color: #333;
  background-color: rgba(0, 0, 0, 0.2);
}

body.dark-theme .app-footer .data-source {
  color: #aaa;
}

body.dark-theme .app-footer .source-link {
  color: #61dafb;
}

body.dark-theme .app-footer .source-link:hover {
  color: #ffffff;
  background-color: #61dafb;
}

body.dark-theme .app-footer .copyright {
  color: #888;
}

/* 淺色主題下的 footer */
body.light-theme .app-footer {
  border-top-color: #ddd;
  background-color: rgba(0, 0, 0, 0.05);
}

body.light-theme .app-footer .data-source {
  color: #555;
}

body.light-theme .app-footer .source-link {
  color: #0366d6;
}

body.light-theme .app-footer .source-link:hover {
  color: #ffffff;
  background-color: #0366d6;
  box-shadow: 0 2px 8px rgba(3, 102, 214, 0.3);
}

body.light-theme .app-footer .copyright {
  color: #666;
}
</style>
