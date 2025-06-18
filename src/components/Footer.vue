<template>
  <footer class="footer">
    <div class="footer-content">
      <div class="footer-info">
        <div class="author">
          <span class="author-text">Stworzone przez</span>
          <strong class="author-name">budziun</strong>
          <span class="year">2025</span>
        </div>
        <div class="project-name">AVG Weather App, made with Vue.js framework</div>
      </div>

      <div class="footer-links">
        <a
            href="https://github.com/budziun"
            target="_blank"
            rel="noopener noreferrer"
            class="github-link profile-link"
            title="Profil GitHub autora"
        >
          <svg class="github-icon" viewBox="0 0 24 24" fill="currentColor">
            <path d="M12 0C5.374 0 0 5.373 0 12 0 17.302 3.438 21.8 8.207 23.387c.599.111.793-.261.793-.577v-2.234c-3.338.726-4.033-1.416-4.033-1.416-.546-1.387-1.333-1.756-1.333-1.756-1.089-.745.083-.729.083-.729 1.205.084 1.839 1.237 1.839 1.237 1.07 1.834 2.807 1.304 3.492.997.107-.775.418-1.305.762-1.604-2.665-.305-5.467-1.334-5.467-5.931 0-1.311.469-2.381 1.236-3.221-.124-.303-.535-1.524.117-3.176 0 0 1.008-.322 3.301 1.23A11.509 11.509 0 0112 5.803c1.02.005 2.047.138 3.006.404 2.291-1.552 3.297-1.23 3.297-1.23.653 1.653.242 2.874.118 3.176.77.84 1.235 1.911 1.235 3.221 0 4.609-2.807 5.624-5.479 5.921.43.372.823 1.102.823 2.222v3.293c0 .319.192.694.801.576C20.566 21.797 24 17.3 24 12c0-6.627-5.373-12-12-12z"/>
          </svg>
          <span class="link-text">Mój Profil</span>
        </a>

        <a
            href="https://github.com/budziun/avg-weather"
            target="_blank"
            rel="noopener noreferrer"
            class="github-link repo-link"
            title="Repozytorium projektu"
        >
          <svg class="github-icon" viewBox="0 0 24 24" fill="currentColor">
            <path d="M9 19c-5 1.5-5-2.5-7-3m14 6v-3.87a3.37 3.37 0 0 0-.94-2.61c3.14-.35 6.44-1.54 6.44-7A5.44 5.44 0 0 0 20 4.77 5.07 5.07 0 0 0 19.91 1S18.73.65 16 2.48a13.38 13.38 0 0 0-7 0C6.27.65 5.09 1 5.09 1A5.07 5.07 0 0 0 5 4.77a5.44 5.44 0 0 0-1.5 3.78c0 5.42 3.3 6.61 6.44 7A3.37 3.37 0 0 0 9 18.13V22"/>
          </svg>
          <span class="link-text">Repozytorium projektu</span>
        </a>
      </div>
    </div>
  </footer>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const footerRef = ref(null)
const isVisible = ref(false)
let observer = null

onMounted(() => {
  // Intersection Observer dla Footer
  observer = new IntersectionObserver(
      (entries) => {
        entries.forEach(entry => {
          if (entry.isIntersecting && !isVisible.value) {
            setTimeout(() => {
              isVisible.value = true
            }, 500)
          }
        })
      },
      {
        threshold: 0.1,
        rootMargin: '10px'
      }
  )

  if (footerRef.value) {
    observer.observe(footerRef.value)
  }
})

onUnmounted(() => {
  if (observer) {
    observer.disconnect()
  }
})
</script>

<style scoped>
.footer {
  background: rgba(45, 52, 54, 0.85);
  color: rgba(255, 255, 255, 0.9);
  padding: 25px 0;
  margin: 32px -32px -32px -32px;
  border-radius: 0 0 20px 20px;
  box-shadow: 0 -5px 20px rgba(0, 0, 0, 0.1);
  border-top: 1px solid rgba(255, 255, 255, 0.1);
}

.footer-content {
  padding: 0 30px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 30px;
}

.footer-info {
  flex: 1;
}

.author {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 5px;
  font-size: 0.95em;
}

.author-text {
  color: rgba(255, 255, 255, 0.7);
}

.author-name {
  font-weight: 600;
  font-size: 1.1em;
}

.year {
  color: rgba(255, 255, 255, 0.8);
  font-weight: 500;
}

.project-name {
  color: rgba(255, 255, 255, 0.6);
  font-size: 0.85em;
  font-style: italic;
}

.footer-links {
  display: flex;
  gap: 15px;
}

.github-link {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 16px;
  background: rgba(255, 255, 255, 0.08);
  border: 1px solid rgba(255, 255, 255, 0.15);
  border-radius: 20px;
  color: rgba(255, 255, 255, 0.9);
  text-decoration: none;
  transition: all 0.3s ease;
  font-weight: 500;
  font-size: 0.9em;
}

.github-link:hover {
  background: rgba(255, 255, 255, 0.15);
  border-color: rgba(255, 255, 255, 0.3);
  transform: translateY(-1px);
  color: white;
}

.profile-link:hover {
  border-color: rgba(116, 185, 255, 0.5);
  background: rgba(116, 185, 255, 0.1);
}

.repo-link:hover {
  border-color: rgba(0, 184, 148, 0.5);
  background: rgba(0, 184, 148, 0.1);
}

.github-icon {
  width: 16px;
  height: 16px;
  transition: transform 0.3s ease;
}

.github-link:hover .github-icon {
  transform: scale(1.1);
}

.link-text {
  font-size: 0.85em;
  white-space: nowrap;
}

/* Responsive design */
@media (max-width: 768px) {
  .footer {
    margin: 30px -20px -20px -20px;
  }

  .footer-content {
    padding: 0 20px;
    flex-direction: column;
    text-align: center;
    gap: 15px;
  }

  .footer-links {
    justify-content: center;
    flex-wrap: wrap;
    gap: 12px;
  }

  .github-link {
    padding: 8px 12px;
    font-size: 0.85em;
  }

  .link-text {
    display: none;
  }

  .github-icon {
    width: 18px;
    height: 18px;
  }
}

@media (max-width: 480px) {
  .footer {
    margin: 20px -15px -15px -15px;
  }

  .footer-content {
    padding: 0 15px;
  }

  .author {
    flex-direction: column;
    gap: 2px;
  }

  .footer-links {
    gap: 10px;
  }
}

/* Accessibility */
@media (prefers-reduced-motion: reduce) {
  .github-link {
    transition: none;
  }
}
</style>
