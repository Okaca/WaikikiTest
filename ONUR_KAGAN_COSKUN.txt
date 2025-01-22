(async () => {
    const init = async () => {
    
        let products = [];
        
        // Initialize localStorage
        const storedProducts = localStorage.getItem('products');
        if (storedProducts) {
            products = JSON.parse(storedProducts);
            
             // Make sure each product has the 'isFavorited' key
            products = products.map(product => ({
                ...product,
                isFavorited: product.isFavorited ?? false // If `isFavorited` is missing, default to false
            }));
            localStorage.setItem('products', JSON.stringify(products));
        } else {
            // Fetch data if not available in localStorage
            products = await fetchData();
            // Add the 'isFavorited' property for each product
            products = products.map(product => ({
                ...product,
                isFavorited: false // Initially set to false
            }));
            localStorage.setItem('products', JSON.stringify(products));
        }
        
        // Inject CSS styles into the document
        buildCSS();
        
        // Create product-details div
        setProductDetailsElement();
        
        // Set products in HTML
        document.querySelector('.product-details').innerHTML = buildHTML(products);
        
        // Initialize carousel
        let currentIndex = 0;
        const productElements = document.querySelectorAll('.carousel .product');
        const totalProducts = productElements.length;
        const visibleProducts = 6.5; // 6.5 products visible at a time
        const productWidth = productElements[0].offsetWidth + 10; // Including margin of 5px on each side
        
        const carouselContainer = document.querySelector('.carousel');
        const maxShift = productWidth * (totalProducts - Math.floor(visibleProducts)); // Maximum shift position

        // Function to update the carousel to show the correct product
        const updateCarousel = () => {
            const shiftAmount = currentIndex * productWidth;
            const maxShiftAmount = Math.min(shiftAmount, maxShift); // Ensure the carousel doesn’t go beyond the last product
            carouselContainer.style.transform = `translateX(-${maxShiftAmount}px)`;
        };

        // Show the first products
        updateCarousel();

        // Handle "Next" button click
        document.querySelector('.carousel-next').addEventListener('click', () => {
            currentIndex = Math.min(currentIndex + 1, totalProducts - Math.floor(visibleProducts));
            updateCarousel();
        });
        
        //Some websites does not incliude jquery, hence I wrote the code with vanilla javascript. Here is how
        //the jquery would be for "Next" button click
        /*
        $('.carousel-next').click(() => {
	    currentIndex = Math.min(currentIndex + 1, totalProducts - Math.floor(visibleProducts));
	    updateCarousel();
	});
	*/

        // Handle "Prev" button click
        document.querySelector('.carousel-prev').addEventListener('click', () => {
            currentIndex = Math.max(currentIndex - 1, 0);
            updateCarousel();
        });
        
        // Handle "Favorite" button click
        document.querySelectorAll('.favorite-heart').forEach(heartButton => {
            heartButton.addEventListener('click', (event) => {
            
                // Get the index from the data-index attribute
                 const index = heartButton.getAttribute('data-index');
        
                // Toggle the favorited class
                heartButton.classList.toggle('favorited');
                
                // Update the isFavorited property for the product
                products[index].isFavorited = heartButton.classList.contains('favorited');

                // Save updated products to localStorage
                localStorage.setItem('products', JSON.stringify(products));

                // Prevent the click from triggering any further actions (like opening a link)
                event.stopPropagation();
            });
        });
        
        // Set initial state for favorite hearts based on isFavorited in products
        // If the code runs second time, favorites are checked from the localStorage
        products.forEach((product, index) => {
            const heartButton = document.querySelector(`.favorite-heart[data-index="${index}"]`);
            if (product.isFavorited) {
                heartButton.classList.add('favorited');
            }
        });
    };

    // Fetch data from github URL
    const fetchData = async () => {
        try {
            const response = await fetch('https://gist.githubusercontent.com/sevindi/5765c5812bbc8238a38b3cf52f233651/raw/56261d81af8561bf0a7cf692fe572f9e1e91f372/products.json');
            if (!response.ok) throw new Error('Network response was not ok');
            return await response.json();
        } catch (error) {
            console.error('Error fetching data:', error);
            return [];
        }
    };
    
    // Create product-details div
    const setProductDetailsElement = () => {
    	let productDetailsDiv = document.querySelector('.product-details');
        if (!productDetailsDiv) {
            productDetailsDiv = document.createElement('div');
            productDetailsDiv.classList.add('product-details');
            document.body.appendChild(productDetailsDiv); 
        }
    }

    const buildHTML = (products) => {
        return `
            <div class="carousel-container">
            <h2 class="carousel-title">You Might Also Like</h2>
                <button class="carousel-prev">Prev</button>
                <button class="carousel-next">Next</button>
                <div class="carousel">
                ${products.map((product, index) => {
                    return `
                        <div class="product">
                            <button class="favorite-heart ${product.isFavorited ? 'favorited' : ''}" data-index="${index}">&#10084;</button> 
                            <a href="${product.url}" class="product-link" target="_blank">
                                <img src="${product.img}" alt="${product.name}" class="product-img">
                                <h2 class="product-name">${product.name}</h2>
                                <p class="product-price">$${product.price.toFixed(2)}</p>
                            </a>
                        </div>
                    `;
                }).join('')}
            </div>
            </div>
        `;
    };
    
    const buildCSS = () => {
        const css = `
	    
            /* Styling for the carousel container */
            carousel-container {
    	        position: relative;
    	        width: 100%;
    	        margin: 20px auto;
    	        overflow: hidden;
    	        display: flex;
    	        flex-direction: column;
    	        justify-content: center;
    	        align-items: center;
            }
            
            /* Styling for the carousel title */
            .carousel-title {
    	        font-size: 24px; /* title size */
    	        font-weight: bold;
    	        text-align: center;
    	        margin-bottom: 20px;
    	        color: #333;
    	    }

            /* Styling for the carousel wrapper */
            .carousel {
                display: flex;
                transition: transform 0.5s ease;
            }

            /* Styling for each individual product */
            .product {
                flex: 0 0 auto;
                margin: 0 5px;
                text-align: center;
                position: relative;
                width: calc(100% / 6.5); /* Show 6.5 products on large screens */
            }

            .product img {
                width: 100%;
                height: auto;
            }

            /* Styling for navigation buttons */
            .carousel-prev, .carousel-next {
                position: relative;
                display: inline-block;
                top: %50;
                background-color: rgba(0, 0, 0, 0.5);
                padding: 10px 15px;
                color: white;
                border: none;
                padding: 10px;
                font-size: 18px;
                cursor: pointer;
                z-index: 10;
                font-size: 18px;
                font-family: 'Arial', sans-serif;
            }

            .carousel-prev {
                float: left;
                margin-left : 0.25em;
            }

            .carousel-next {
                float: right;
                margin-right : 0.25em;
            }

            .carousel-prev:hover, .carousel-next:hover {
                background-color: rgba(0, 0, 0, 0.7);
            }
            
            /* Styling for the heart button */
            .favorite-heart {
                position: absolute;
                top: 10px;
                right: 10px;
                font-size: 24px; /* heart size */
                color: #ccc; /* Default color*/
                background: none;
                border: none;
                cursor: pointer;
                transition: color 0.3s ease;
            }

            /* Heart turned blue when clicked */
            .favorite-heart.favorited {
                color: #006cb7; /* Waikiki Blue */
            }
        `;

        // Create a <style> element to hold the CSS
        const styleSheet = document.createElement("style");
        styleSheet.type = "text/css";
        styleSheet.innerText = css;

        // Append the <style> element to the head of the document
        document.head.appendChild(styleSheet);
    };

    await init();
})();
