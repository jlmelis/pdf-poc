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

  async function printPDF() {
    console.log('printing');
    const printContainer = document.getElementById('printContainer');
    const pages = document.querySelectorAll('.pageCanvas');

    for (const page of pages) {
      const image = new Image();
      image.src = await resizedataURL(page.toDataURL("image/png"), page.width * 2, page.height * 2);
      image.classList.add('pageImg');
      image.alt = 'pdf page';
      printContainer.appendChild(image);
    }

    window.print();
    removeAllChildNodes(printContainer);
  }

  function removeAllChildNodes(parent) {
    while (parent.firstChild) {
        parent.removeChild(parent.firstChild);
    }
  }

  // Takes a data URI and returns the Data URI corresponding to the resized image at the wanted size.
function resizedataURL(datas, wantedWidth, wantedHeight){
    return new Promise(async function(resolve,reject){

        // We create an image to receive the Data URI
        var img = document.createElement('img');

        // When the event "onload" is triggered we can resize the image.
        img.onload = function()
        {        
            // We create a canvas and get its context.
            var canvas = document.createElement('canvas');
            var ctx = canvas.getContext('2d');

            // We set the dimensions at the wanted size.
            canvas.width = wantedWidth;
            canvas.height = wantedHeight;

            // We resize the image with the canvas method drawImage();
            ctx.drawImage(this, 0, 0, wantedWidth, wantedHeight);

            var dataURI = canvas.toDataURL();

            // This is the return of the Promise
            resolve(dataURI);
        };

        // We put the Data URI in the image's src attribute
        img.src = datas;

    })
}// Use it like : var newDataURI = await resizedataURL('yourDataURIHere', 50, 50);
</script>

<main>
  <button id="pdfButton" class="noPrint" on:click={loadPDF}>Load pdf</button>
  <div class="modal noPrint" class:is-active={showPDF}>
    <div class="modal-background noPrint" on:click={() => {showPDF = false}}></div>
    <div class="modal-card noPrint">
      <header  class="modal-card-head">
        <button class="button" on:click="{printPDF}">Print</button>
        <div class="modal-card-title">PDF</div>
        <button class="delete" aria-label="close" on:click={() => {showPDF = false}}></button>
      </header>
      <section class="modal-card-body">
        <div id="pdfContainer"  bind:this={pdfContainer}></div>
      </section>
    </div>
  </div>
  <div id="printContainer" ></div>
</main>

<style>

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}

  @media print {
    .noPrint {
      display: none;
    }

    #printContainer {
      display: block;
    }

    @page {
      margin:0cm;
    }
  }

  #printContainer {
    display: hidden;
  }

  #pdfButton {
    position: fixed;
    top: 0;
    left: 0;
  }

</style>