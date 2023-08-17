<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";

    // Function to introduce a delay during sorting animations
    function wait() {
        const speeds = [1000, 500, 50]; // Fast, Medium, Slow speeds in milliseconds
        
        return new Promise((resolve) =>
            setTimeout(resolve, speeds[sortingSpeed - 1])
        );
    }

    // Function to set the sorting speed level
    function setSortingSpeed(level) {
        sortingSpeed = level;
    }

    // Function to visualize the sorting process
    async function visualizeSort(data) {
        const bars = svg.selectAll(".bar").data(data);

        bars
            .transition()
            .duration(300) // Duration of the transition animation
            .attr("y", (d) => 300 - d * 3)
            .attr("height", (d) => d * 3)
            .attr("fill", (d, i) => (highlightedBars.has(i) ? "#e74c3c" : "#3498db"));
            // Reset bar colors if sorting is not in progress
        if (sortingInProgress === false) {
            svg.selectAll(".bar").attr("fill", "#008000"); // Green color
        }
    }

    // Default sorting speed level (medium)
    let sortingSpeed = 2;

    // Default selected sorting algorithm
    let selectedAlgorithm = "bubble";

    // Flag to indicate if sorting is in progress
    let sortingInProgress = false;

    // Array of data to be sorted
    let dataToSort = [];

    // Default array size
    let arraySize = 20;

    // Get the container element for bars
    let barContainer = document.getElementById("barContainer");

    // Function to generate a random array of data
    function generateRandomArray() {
        dataToSort = Array.from(
            { length: arraySize },
            () => Math.floor(Math.random() * 100) + 1
        );

        updateVisualization();

        const bars = document.querySelectorAll(".bar");
        bars.forEach((bar) => {
            bar.style.backgroundColor = "#0051ff";
        });

        // Add the gray background color and border
        barContainer.classList.add("with-background");
    }

    let data = [
        /* array of data */
    ];

    // Initialize variables
    let svg; // SVG element
    let xScale; // X-axis scale
    let barWidth = 1; // Width of each bar
    let highlightedBars = new Set(); // Set of indices of highlighted bars
    
    // Triggered when the component is mounted
    onMount(() => {
        const container = d3.select(".visualization-container");
        const maxWidth = 1; // Placeholder for maximum width

        svg = container.append("svg").attr("width", maxWidth).attr("height", 300);
        // Add the background rectangle
        svg
            .append("rect")
            .attr("class", "chart-background")
            .attr("width", maxWidth)
            .attr("height", 300)
            .attr("fill", "#f1f1f1"); // Set the gray background color

        // Adjust the scale based on the number of elements and viewport width
        xScale = d3
            .scaleBand()
            .domain(data.map((_, i) => i))
            .range([0, Math.min(maxWidth, data.length * 30)]) // Adjust the multiplier for desired width
            .padding(0.1);

        barWidth = xScale.bandwidth();

        // Update the visualization
        updateVisualization();

        // Color the bars that are already sorted
        colorBarsCompleted();
    });
    // Function to update the visualization based on data and container width
    function updateVisualization() {
        const container = d3.select(".visualization-container");
        const containerWidth = container.node().clientWidth;

        // Calculate the maximum number of bars that can fit within the container width
        const maxBarsToFit = Math.floor(containerWidth / barWidth);

        // Adjust the array size if needed to fit within the container
        if (data.length > maxBarsToFit) 
            data = data.slice(0, maxBarsToFit);

        // Update xScale and barWidth based on the adjusted data length
        xScale = d3
            .scaleBand()
            .domain(data.map((_, i) => i))
            .range([0, containerWidth])
            .padding(0.1);

        // barWidth = xScale.bandwidth();

        // Update bars
        const bars = svg.selectAll(".bar").data(data);
        // Enter new bars
        bars
            .enter()
            .append("rect")
            .attr("class", "bar")
            .attr("x", (d, i) => xScale(i))
            .attr("y", (d) => 300 - d * 3)
            .attr("width", barWidth)
            .attr("height", (d) => d * 3)
            .attr("fill", (d, i) => (highlightedBars.has(i) ? "#e74c3c" : "#3498db"));
        // Remove any excess bars
            bars.exit().remove();
        // Remove the gray background color and border if needed
        if (dataToSort.length === 0) 
            barContainer.classList.remove("with-background");
    }

    // Function to display algorithm details modal
    function showMoreDetails() {
        const modal = document.getElementById("algorithmDetailsModal");
        const closeModal = document.getElementById("closeModal");
        const algorithmDetailsText = document.getElementById(
            "algorithmDetailsText"
        );

        // Set algorithm details based on the selected algorithm
        if (selectedAlgorithm === "bubble") 
            algorithmDetailsText.textContent =
            "Bubble Sort works by repeatedly comparing adjacent elements in a list from left to right,\n" + 
            "swapping them if one is larger or smaller than the other until there are no more elements to swap\n" +
            "and the array is sorted.\n\n" +
            "Worst-case time complexity: O(n^2)";
        else if (selectedAlgorithm === "insertion")
            algorithmDetailsText.textContent =
            "Insertion Sort works by taking one unsorted element and iteratively comparing it to the sorted\n" +
            "elements in the array, inserting it where it is larger or smaller than another element until\n" +
            "there are no more unsorted elements.\n\n" +
            "Worst-case time complexity: O(n^2)";
        else if (selectedAlgorithm === "merge")
            algorithmDetailsText.textContent =
            "Merge Sort works by recursively dividing the array into smaller subarrays, sorting each subarray\n" +
            "and then merging the sorted subarrays into a complete sorted array.\n\n" +
            "Worst-case time complexity: O(nlogn)";
        else if (selectedAlgorithm === "quick")
            algorithmDetailsText.textContent =
            "Quick Sort works by selecting a pivot element and splitting the array into two parts.\n" +
            "The elements smaller than the pivot are moved to the left of the pivot element while the larger elements\n" +
            "are moved to the right of the pivot element. This process is repeated until the array is completely sorted.\n\n" +
            "Worst-case time complexity: O(nlogn)";
        else if (selectedAlgorithm === "heap")
            algorithmDetailsText.textContent =
            "Heap sort works by converting the array into a binary heap, which is a tree-based data structure.\n" + 
            "The root element is swapped with the last element in the array and the last element is removed\n" + 
            "from the heap. Heapify the elements in the array and repeat until a max or min heap is obtained.\n\n" +
            "Worst-case time complexity: O(nlogn)";

        // Display the modal
        modal.style.display = "block";

        // behavior for closing the modal
        closeModal.onclick = function () {
            modal.style.display = "none";
        };

        // Close the modal when clicking outside of it
        window.onclick = function (event) {
            if (event.target === modal) 
            modal.style.display = "none";
        };
    }

    // Function to color completed bars
    function colorBarsCompleted() {
        svg.selectAll(".bar").classed("completed", true); // Add the completed class to the bars
    }

    // Function to handle algorithm selection   
    function handleAlgorithmSelection(event) {
        selectedAlgorithm = event.target.value; // Update the existing selectedAlgorithm variable

        // Get the element for displaying algorithm description
        const descriptionParagraph = document.getElementById(
            "algorithmDescription"
        );

        // Initialize an empty string to store the description
        let description = "";
        
        // Determine the description based on the selected algorithm
        if (selectedAlgorithm === "bubble") {
            description =
            "Bubble Sort works by repeatedly comparing adjacent elements in a list from left to right,\n" + 
            "swapping them if one is larger or smaller than the other until there are no more elements to swap\n" +
            "and the array is sorted.\n\n" +
            "Worst-case time complexity: O(n^2)";
        } else if (selectedAlgorithm === "insertion") {
            description =
            "Insertion Sort works by taking one unsorted element and iteratively comparing it to the sorted\n" +
            "elements in the array, inserting it where it is larger or smaller than another element until\n" +
            "there are no more unsorted elements.\n\n" +
            "Worst-case time complexity: O(n^2)";
        } else if (selectedAlgorithm === "merge") {
            description =
            "Merge Sort works by recursively dividing the array into smaller subarrays, sorting each subarray\n" +
            "and then merging the sorted subarrays into a complete sorted array.\n\n" +
            "Worst-case time complexity: O(nlogn)";
        } else if (selectedAlgorithm === "quick") {
            description =
            "Quick Sort works by selecting a pivot element and splitting the array into two parts.\n" +
            "The elements smaller than the pivot are moved to the left of the pivot element while the larger elements\n" +
            "are moved to the right of the pivot element. This process is repeated until the array is completely sorted.\n\n" +
            "Worst-case time complexity: O(nlogn)";
        } else if (selectedAlgorithm === "heap") {
            description =
            "Heap sort works by converting the array into a binary heap, which is a tree-based data structure.\n" + 
            "The root element is swapped with the last element in the array and the last element is removed\n" + 
            "from the heap. Heapify the elements in the array and repeat until a max or min heap is obtained.\n\n" +
            "Worst-case time complexity: O(nlogn)";
        }

        // Set the description text in the HTML element
        descriptionParagraph.textContent = description;

        // Add your implementation here based on the selected algorithm
        console.log("Selected Algorithm:", selectedAlgorithm);
    }
    // Function to initiate sorting
    async function startSorting() {
        // Check if there's data to sort and an algorithm is selected
        if (dataToSort.length === 0) {
            alert("Please generate an array and select an algorithm before clicking Start.");
            return;
        }

        if (!selectedAlgorithm) {
            alert("Please select an algorithm before clicking Start.");
            return;
        }

        sortingInProgress = true;

        if (selectedAlgorithm === "bubble") await bubbleSortWithVisualization();
        else if (selectedAlgorithm === "heap") await heapSortWithVisualization();
        else if (selectedAlgorithm === "merge") await mergeSortWithVisualization();
        else if (selectedAlgorithm === "insertion") await insertionSortWithVisualization(dataToSort);
        else if (selectedAlgorithm === "quick") await quickSortWithVisualization(dataToSort, 0, dataToSort.length - 1);

        sortingInProgress = false;

        console.log("Sorting completed with algorithm:", selectedAlgorithm);

        // Change the color of bars to indicate sorting completion
        const bars = document.querySelectorAll(".bar");

        bars.forEach((bar) => {
            bar.style.backgroundColor = "#4BB543";
        });
    }
    // Function to apply random colors to bars with a delay
    async function applyRandomColorsWithDelay(bars, delay) {
        const colorPalette = [
            "#FF5733",
            "#FFC300",
            "#C70039",
            "#900C3F",
            "#3C1874",
            "#00A6ED",
            "#34A853",
            "#FBBC05",
            "#F94877",
            "#4285F4",
            "#FF6D00",
            "#2196F3",
            "#FF4500",
            "#9C27B0",
            "#FFD700",
            "#FF5722",
            "#795548",
            "#4CAF50",
            "#607D8B",
            "#E91E63",
            "#00BCD4",
            "#FF9800",
            "#8A2BE2",
            "#FFA500",
            "#32CD32",
            "#FF00FF",
            "#ADFF2F",
            "#FF1493",
            "#00FF7F",
            "#8B008B",
            "#FFFF00",
            "#008000",
            "#9932CC",
            "#F0E68C",
            "#00FF00",
            "#BA55D3",
            "#7CFC00",
            "#800080",
            "#00FF00",
            "#DDA0DD",
            "#228B22",
            "#FF00FF",
            "#ADFF2F",
            "#FF4500",
            "#2E8B57",
            "#FF69B4",
            "#008B8B",
            "#FF6347",
            "#2F4F4F",
            "#DC143C",
            "#006400",
            "#DAA520",
            "#20B2AA",
            "#CD5C5C",
            "#6B8E23",
            "#D2691E",
            "#48D1CC",
            "#8B4513",
            "#40E0D0",
            "#A0522D",
            "#1E90FF",
            "#FF8C00",
            "#556B2F",
            "#B22222",
            "#8B0000",
            "#B8860B",
            "#8B4513",
            "#8B4513",
            "#ADFF2F",
            "#B8860B",
            "#8B4513",
            "#8B4513",
            "#ADFF2F",
            "#8B4513",
            "#8B4513",
            "#ADFF2F",
            "#8B4513",
            "#8B4513",
            "#ADFF2F",
            "#8B4513",
            "#8B4513",
            "#ADFF2F",
            "#8B4513",
            "#8B4513",
            "#ADFF2F",
            "#8B4513",
            "#8B4513",
            "#ADFF2F",
            "#8B4513",
            "#8B4513",
            "#ADFF2F",
            "#8B4513",
            "#8B4513",
            "#ADFF2F",
            "#8B4513",
            "#8B4513",
            "#ADFF2F",
            "#8B4513",
            "#8B4513",
            "#ADFF2F",
            "#8B4513",
            "#8B4513",
            "#ADFF2F",
            "#8B4513",
            "#8B4513",
            "#ADFF2F",
            "#8B4513",
            "#8B4513",
            "#ADFF2F",
            "#8B4513",
            "#8B4513",
            "#ADFF2F",
            "#8B4513",
            "#8B4513",
            "#ADFF2F",
            "#8B4513",
            "#8B4513",
            "#ADFF2F",
            "#8B4513",
            "#8B4513",
            "#ADFF2F",
        ];

        // Loop through each bar and apply a random color
        for (const bar of bars) {
            // Generate a random index to select a color from the palette
            // Set the background color of the bar to the selected random color
            bar.style.backgroundColor = colorPalette[Math.floor(Math.random() * colorPalette.length)];

            // Wait for the specified delay before proceeding to the next bar
            await wait(delay);
        }
    }

    // Function to perform Bubble Sort with visualization
    async function bubbleSortWithVisualization() {
        const bars = document.querySelectorAll(".bar");

        // Loop through the array for bubble sort iterations
        for (let i = 0; i < dataToSort.length - 1; i++) {
            for (let j = 0; j < dataToSort.length - i - 1; j++) 
                // If adjacent elements are out of order, swap them
                if (dataToSort[j] > dataToSort[j + 1]) 
                    await swapWithVisualization(j, j + 1); // Call the swap function with visualization

            await applyRandomColorsWithDelay(bars, 500); // Apply random colors with 0.5-second delay
        }

        // After the sorting process is completed, change the color of all bars to blue
        bars.forEach((bar) => {
            bar.style.backgroundColor = "#4BB543";
        });
    }

    // Function to perform a swap with visualization
    async function swapWithVisualization(i, j) {
        await wait(300); // Adjust the delay time as needed

        let temp = dataToSort[i];
        dataToSort[i] = dataToSort[j];
        dataToSort[j] = temp;
        
        // Highlight the bars being swapped
        svg
            .selectAll(".bar")
            .filter((d, index) => index === i || index === j)
            .classed("highlight", true); // Apply the highlight class

        // Update visualization and remove the highlight after animation frame
        requestAnimationFrame(() => {
            visualizeSort(dataToSort); // Update the visualization
            svg
            .selectAll(".bar")
            .filter((d, index) => index === i || index === j)
            .classed("highlight", false); // Remove the highlight class
        });
    }

    // Function to perform Heap Sort with visualization
    async function heapSortWithVisualization() {
        const n = dataToSort.length;
        const bars = document.querySelectorAll(".bar"); // Get all bar elements

        // Build max heap
        for (let i = Math.floor(n / 2) - 1; i >= 0; i--) 
            await heapifyWithVisualization(n, i); // Call heapify function with visualization

        // Extract elements one by one
        for (let i = n - 1; i > 0; i--) {
            await swapWithVisualization(0, i); // Swap root with the last element
            await heapifyWithVisualization(i, 0); // Call heapify function with visualization

            await applyRandomColorsWithDelay(bars, 500); // Apply random colors with 0.5-second delay
        }

        // After the sorting process is completed, change the color of all bars to blue
        bars.forEach((bar) => {
            bar.style.backgroundColor = "#4BB543";
        });
    }

    // Function to perform Insertion Sort with visualization
    async function insertionSortWithVisualization() {
        console.log("Starting insertion sort...");
        const bars = document.querySelectorAll(".bar"); // Get all bar elements

        for (let i = 1; i < dataToSort.length; i++) {
            let key = dataToSort[i];
            let j = i - 1;

            // Move elements of dataToSort[0..i-1] that are greater than key
            // to one position ahead of their current position
            while (j >= 0 && dataToSort[j] > key) {
                dataToSort[j + 1] = dataToSort[j];
                j--;
            }

            dataToSort[j + 1] = key;

            // Update visualization and apply random colors with a delay
            visualizeSort(dataToSort); // Update the visualization after each step

            await applyRandomColorsWithDelay(bars, 500); // Apply random colors with 0.5-second delay

            await wait(500); // Add an extra delay as needed
        }

        // After sorting, change the color of all bars to indicate completion
        bars.forEach((bar) => {
            bar.style.backgroundColor = "#4BB543"; // Green color
        });
    }

    // Function to perform heapify with visualization
    async function heapifyWithVisualization(n, i) {
        let largest = i;
        let left = 2 * i + 1;
        let right = 2 * i + 2;

        // Find the largest among the root, left child, and right child
        if (left < n && dataToSort[left] > dataToSort[largest]) 
            largest = left;

        if (right < n && dataToSort[right] > dataToSort[largest]) 
            largest = right;

        // If the largest is not the root, swap with the largest and heapify the affected subtree
        if (largest !== i) {
            await swapWithVisualization(i, largest);
            await heapifyWithVisualization(n, largest);
        }
    }

    // Function to perform Merge Sort with visualization
    async function mergeSortWithVisualization() {
        const bars = document.querySelectorAll(".bar");
        const colorPalette = [
            "#FF5733",
            "#FFC300",
            "#C70039",
            "#900C3F",
            "#3C1874",
            "#00A6ED",
            "#FF5733",
            "#FFC300",
            "#C70039",
            "#900C3F",
            "#3C1874",
            "#00A6ED",
            "#FF5733",
            "#FFC300",
            "#C70039",
            "#900C3F",
        ];

        // Apply random colors before starting merge sort
        await applyRandomColorsWithDelay(bars, colorPalette, 500);

        // Start merge sort algorithm
        await mergeSort(0, dataToSort.length - 1, bars, colorPalette);
    }

    // Recursive function to perform Merge Sort
    async function mergeSort(low, high, bars, colorPalette) {
        if (low < high) {
            const mid = Math.floor((low + high) / 2);

            // Recursively divide and sort the two halves
            await mergeSort(low, mid, bars, colorPalette);
            await mergeSort(mid + 1, high, bars, colorPalette);

            // Merge the sorted halves
            await mergeArrays(low, mid, high, bars, colorPalette);
        }
    }

    // Function to merge two sorted arrays while visualizing the process
    async function mergeArrays(low, mid, high, bars, colorPalette) {
        // Create left and right subarrays
        const leftArr = dataToSort.slice(low, mid + 1);
        const rightArr = dataToSort.slice(mid + 1, high + 1);

        let i = 0, j = 0, k = low;

        // Merge the two subarrays into the original array
        while (i < leftArr.length && j < rightArr.length) {
            if (leftArr[i] <= rightArr[j]) {
                dataToSort[k] = leftArr[i];
                i++;
            } else {
                dataToSort[k] = rightArr[j];
                j++;
            }

            k++;

            await wait(500); // Delay to visualize each step
        }

        // Append any remaining elements in leftArr
        while (i < leftArr.length) {
            dataToSort[k] = leftArr[i];
            i++; k++;

            await wait(500); // Delay to visualize each step
        }

        // Append any remaining elements in rightArr
        while (j < rightArr.length) {
            dataToSort[k] = rightArr[j];
            j++; k++;

            await wait(500); // Delay to visualize each step
        }
    }

    // Function to perform Quick Sort with visualization
    async function quickSortWithVisualization(arr, low, high) {
        if (low < high) {
            const partitionIndex = await partitionWithVisualization(arr, low, high);

            await wait(300); // Wait for visualization to complete
            await quickSortWithVisualization(arr, low, partitionIndex - 1);

            await wait(300); // Wait for visualization to complete
            await quickSortWithVisualization(arr, partitionIndex + 1, high);
        }
    }


    // Function to partition the array for Quick Sort with visualization
    async function partitionWithVisualization(arr, low, high) {
        const pivot = arr[high];
        let i = low - 1;

        // Iterate through the array and move elements smaller than the pivot to the left
        for (let j = low; j <= high - 1; j++) 
            if (arr[j] < pivot) {
                i++;
                await swapWithVisualization(i, j, arr); // Visualize the swapping step
            }

        await swapWithVisualization(i + 1, high, arr); // Visualize the final swap with the pivot

        return i + 1; // Return the index of the pivot after partitioning
    }
</script>

<!-- Preconnect to Google Fonts for improved performance -->
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />

<!-- Include the fonts for the application -->
<link
  href="https://fonts.googleapis.com/css2?family=Cabin:wght@500&family=Exo&display=swap"
  rel="stylesheet"
/>

<!-- Include the external stylesheet for additional styling -->
<link rel="stylesheet" href="/styles.css" />

<!-- Main heading of the application -->
<h1 id="main-heading">Sorting Algorithm Visualizer</h1>

<!-- Introduction text explaining the purpose of the application -->
<p id="intro-text">
  Welcome to the Sorting Algorithm Visualizer! This interactive web application
  allows you to explore and understand various sorting algorithms by visualizing
  their sorting process. Choose an algorithm, set the array size, and watch as
  the sorting process unfolds in real-time. Whether you're a coding enthusiast
  or just curious about how sorting works, this tool is designed to help you
  learn and have fun!
  Choose between bubble, insertion, merge, quick, and heap sort to visualize various sorting array
  algorithms and enhance your understanding of this fundamental concept in programming.
</p>

<!-- Button to show more details about the selected algorithm -->
<div class="con">
  <button class="start-button" on:click={showMoreDetails}>More Details</button>
  
  <!-- Dropdown menu to select the sorting algorithm -->
  <div class="select-container">
    <select style="width: 10rem;" id="algorithmSelect" on:change={handleAlgorithmSelection}>
      <option value="bubble">Bubble Sort</option>
      <option value="insertion">Insertion Sort</option>
      <option value="merge">Merge Sort</option>
      <option value="quick">Quick Sort</option>
      <option value="heap">Heap Sort</option>
    </select>
  </div>
  
  <!-- Button to start the sorting process -->
  <div class="button-container">
    <button class="start-button" on:click={startSorting}>Start</button>
  </div>
</div>

<!-- Modal to display more details about the selected algorithm -->
<div class="modal" id="algorithmDetailsModal">
  <div class="modal-content">
    <span class="close" id="closeModal">&times;</span>
    <p id="algorithmDetailsText" />
  </div>
</div>

<!-- Range input to adjust the array size -->
<div class="range-container">
  <label for="arraySize" class="intro-text">Array Size: {arraySize}</label>
  <input
    type="range"
    id="arraySize"
    min="5"
    max="100"
    step="1"
    bind:value={arraySize}
  />

  <!-- Button to generate a new random array -->
  <div class="button-container">
    <button class="start-button" on:click={generateRandomArray}>Generate New Array</button>
  </div>

  <!-- Range input to adjust the sorting speed -->
  <label for="sortingSpeed" class="intro-text">Sorting Speed:</label>
  <input
    type="range"
    id="sortingSpeed"
    min="1"
    max="3"
    step="1"
    bind:value={sortingSpeed}
  />
</div>

<!-- Container for displaying the bars representing the array elements -->
<div class="bar-container {dataToSort.length > 0 ? 'with-background' : ''}" id="barContainer">
  {#each dataToSort as value}
    <div class="bar" id="bar" style="height: {value * 3}px; width: 1px;" />
  {/each}
</div>
