<script>
  import pdfjs from 'pdfjs-dist';
  import pdfjsWorkerEntry from "pdfjs-dist/build/pdf.worker.entry";
  import printJS from 'print-js';

  pdfjs.GlobalWorkerOptions.workerSrc = pdfjsWorkerEntry;

  let showPDF = false;
  let pdfContainer;
  let currPage = 1;
  let numPages = 0;
  let thePDF = null;


  async function loadPDF() {
    const loadingTask = pdfjs.getDocument("sample.pdf");
    thePDF = await loadingTask.promise;

    numPages = thePDF.numPages;
    thePDF.getPage( 1 ).then( handlePages );

    showPDF = true
  }

  function handlePages(page) {
    const viewport = page.getViewport({scale: 1.0});

    // Apply page dimensions to the <canvas> element.
    const canvas = document.createElement("canvas");
    canvas.classList.add('pageCanvas');
    const context = canvas.getContext("2d");
    canvas.height = viewport.height;
    canvas.width = viewport.width;

    // Render the page into the <canvas> element.
    const renderContext = {
      canvasContext: context,
      viewport: viewport
    };
    page.render(renderContext);

    pdfContainer.appendChild(canvas);

    //move to next page
    currPage++;
    if ( thePDF !== null && currPage <= numPages )
    {
        thePDF.getPage( currPage ).then( handlePages );
    }
  }

  function printPDF() {
    console.log('printing');
    //printJS('sample.pdf');
    window.print();
    // var iframe = document.createElement('iframe');
    // iframe.src = 'sample.pdf';
    // iframe.style.cssText = "display: none; height: 100%; width: 100%";

    // document.body.appendChild(iframe);

    // iframe.focus();
    // iframe.contentWindow.print();
  }
</script>

<main>
  <button id="pdfButton" class="noPrint" on:click={loadPDF}>Load pdf</button>
  <!-- <div class="modal" class:is-active={showPDF}>
    <div class="modal-background" on:click={() => {showPDF = false}}></div>
    <div class="modal-card">
      <header  class="modal-card-head noPrint">
        <button class="button" on:click="{printPDF}">Print</button>
        <div class="modal-card-title">PDF</div>
        <button class="delete" aria-label="close" on:click={() => {showPDF = false}}></button>
      </header>
      <section class="modal-card-body">
        <div id="pdfContainer"  bind:this={pdfContainer}></div>
      </section>
    </div>
  </div> -->
  <button class="button noPrint" on:click="{printPDF}">Print</button>
  <div id="pdfContainer"  bind:this={pdfContainer}></div>
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

  @media print {
    .noPrint {
      display: none;
    }
    .pageCanvas {
      
      position: absolute;
      left: 0;
      top: 0;
      right: 0;
      bottom: 0;
      width: 100%;
    }
    #pdfContainer {
      max-width: 100%;
    }

  }



  #pdfButton {
    position: fixed;
    top: 0;
    left: 0;
  }

</style>