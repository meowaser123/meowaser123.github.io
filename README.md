
                    }
                    
                    // For block elements, center them
                    if (window.getComputedStyle(el).display === 'block') {
                        el.style.marginLeft = 'auto';
                        el.style.marginRight = 'auto';
                    }
                });
                
                // Force center all headings
                const renderedHeadings = document.querySelectorAll('.blog-content h1, .blog-content h2, .blog-content h3, .blog-content h4, .blog-content h5, .blog-content h6');
                renderedHeadings.forEach(heading => {
                    heading.style.textAlign = 'center';
                    heading.style.width = '100%';
                    
                    // Special styling for "Raid Bosses" heading
                    if (heading.textContent.trim().includes('Raid Bosses')) {
                        heading.style.color = '#FF9D43';
                        heading.style.fontSize = '2rem';
                    }
                });
                
                // Force center all images
                const renderedImages = document.querySelectorAll('.blog-content img');
                renderedImages.forEach(img => {
                    img.style.display = 'block';
                    img.style.margin = '20px auto';
                    img.style.maxWidth = '100%';
                });
            }, 100);
        }
        
        // Function to show a message when no content is available
        function showNoContentMessage() {
            const contentElement = document.getElementById('content');
            
            contentElement.innerHTML = `
                <div class="error-container">
                    <p>No new devblog available at the moment.</p>
                    <p>Please check back later for updates!</p>
                    <button class="refresh-button" onclick="fetchLatestDevblog()">Check Again</button>
                </div>
            `;
        }
        
        // Helper function to format date
        function formatDate(dateString) {
            if (!dateString) return '';
            
            const options = { year: 'numeric', month: 'long', day: 'numeric' };
            return new Date(dateString).toLocaleDateString(undefined, options);
        }
        
        // Initial fetch when page loads
        document.addEventListener('DOMContentLoaded', fetchLatestDevblog);
    </script>
</body>
</html>
