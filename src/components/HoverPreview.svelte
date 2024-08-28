<script>
    import { onMount } from 'svelte';
  
    let preview = { visible: false, content: null, position: { x: 0, y: 0 } };
    let timeoutId;
  
    onMount(() => {
      document.body.addEventListener('mouseover', handleMouseEnter);
      document.body.addEventListener('mouseout', handleMouseLeave);
  
      return () => {
        document.body.removeEventListener('mouseover', handleMouseEnter);
        document.body.removeEventListener('mouseout', handleMouseLeave);
      };
    });
  
    function handleMouseEnter(event) {
      const link = event.target.closest('a[href^="/"]');
      if (!link) return;
  
      clearTimeout(timeoutId);
      timeoutId = setTimeout(() => {
        fetchPreview(link.getAttribute('href')).then((content) => {
          preview = {
            visible: true,
            content,
            position: { x: event.pageX, y: event.pageY }
          };
        });
      }, 300);
    }
  
    function handleMouseLeave() {
      clearTimeout(timeoutId);
      preview.visible = false;
    }
  
    async function fetchPreview(url) {
      try {
        const response = await fetch(url);
        const html = await response.text();
        const parser = new DOMParser();
        const doc = parser.parseFromString(html, 'text/html');
        
        const title = doc.querySelector('title').textContent;
        const firstHeading = doc.querySelector('h1, h2, h3, h4, h5, h6');
        const footer = doc.querySelector('footer');
  
        let content = '';
        let currentNode = firstHeading;
  
        while (currentNode && currentNode !== footer) {
          if (currentNode.nodeType === Node.TEXT_NODE) {
            content += currentNode.textContent.trim() + ' ';
          } else if (currentNode.nodeType === Node.ELEMENT_NODE && !['script', 'style'].includes(currentNode.tagName.toLowerCase())) {
            content += currentNode.textContent.trim() + ' ';
          }
          currentNode = currentNode.nextSibling;
        }
  
        return { title, content: content.slice(0, 200) + (content.length > 200 ? '...' : '') };
      } catch (error) {
        console.error('Error fetching preview:', error);
        return null;
      }
    }
  </script>
  
  {#if preview.visible && preview.content}
    <div 
      class="preview"
      style="top: {preview.position.y + 20}px; left: {preview.position.x + 20}px;"
    >
      <h3>{preview.content.title}</h3>
      <p>{preview.content.content}</p>
    </div>
  {/if}
  
  <style>
    .preview {
      position: absolute;
      z-index: 1000;
      background-color: white;
      border: 1px solid #ccc;
      border-radius: 4px;
      padding: 10px;
      max-width: 300px;
      box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    }
    h3 {
      margin: 0 0 10px 0;
      font-size: 16px;
    }
    p {
      margin: 0;
      font-size: 14px;
    }
  </style>