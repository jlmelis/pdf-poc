<script>
  import pdfjs from 'pdfjs-dist';
  import pdfjsWorkerEntry from "pdfjs-dist/build/pdf.worker.entry";

  pdfjs.GlobalWorkerOptions.workerSrc = pdfjsWorkerEntry;

  let pdfContainer;

  async function loadPDF() {
    
    const loadingTask = pdfjs.getDocument("sample.pdf");
    const pdf = await loadingTask.promise;

    // Load information from the first page.
    const page = await pdf.getPage(1);

    const scale = 1;
    const viewport = page.getViewport({scale: 1.0});


    // Apply page dimensions to the <canvas> element.
    const canvas = document.createElement("canvas");
    const context = canvas.getContext("2d");
    canvas.height = viewport.height;
    canvas.width = viewport.width;

    // Render the page into the <canvas> element.
    const renderContext = {
      canvasContext: context,
      viewport: viewport
    };
    await page.render(renderContext);

    pdfContainer.appendChild(canvas);

    console.log("Page rendered!");
  }

  function handlePages(page) {
    
  }
</script>

<main>
  <button id="pdfButton" on:click={loadPDF}>Load pdf</button>
  <div bind:this={pdfContainer}>

  </div>
  <!-- <canvas id="pdf"></canvas> -->
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}

  #pdfButton {
    position: fixed;
    top: 0;
    left: 0;
  }
</style>