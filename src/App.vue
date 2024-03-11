<!-- @format -->

<template>
  <div style="display: flex">
    <div class="container">
      <!-- Left side -->
      <div class="left-side">
        <h3 class="labelheading">Layout</h3>
        <div class="labeltype">
          <div class="label">Width</div>
          <input type="number" v-model="sliderWidth" class="icon_input" />
        </div>
        <div class="labeltype">
          <div class="label">Height</div>
          <input type="number" v-model="sliderHeight" class="icon_input" />
        </div>
        <div class="labeltype">
          <div class="label">Background-color</div>
          <input type="color" v-model="backgroundcolor" />
        </div>
        <button @click="handleLayout" class="button">Apply</button>
      </div>

      <div>
        <div class="labeltype">
          <div class="label">Previous Icon</div>
          <div>
            <input type="text" v-model="Previousicontext" class="icon_input" />
            <button class="pull-left topBarButtons" @click="openFilePrevInput">
              <i class="fa-solid fa-file"></i>
            </button>
            <input type="file" @change="handlePrevIconChange" style="display: none" id="handlePrevIcon" />
          </div>
        </div>

        <div class="labeltype">
          <div class="label">Next Icon</div>
          <div>
            <input type="text" v-model="Nexticontext" class="icon_input" />
            <button class="pull-left topBarButtons" @click="openFileNextInput">
              <i class="fa-solid fa-file"></i>
            </button>
            <input type="file" @change="handleNextIconChange" style="display: none" id="handleNextIcon" />
          </div>
        </div>
        <div class="labeltype">
          <div class="label">Margin XY</div>
          <div>
            <input type="number" v-model="marginX" class="icon_input" style="width: 33%" />
            <input type="number" v-model="marginY" class="icon_input" style="width: 33%" />
          </div>
        </div>
        <div class="labeltype">
          <div class="label">Width-Height</div>
          <div>
            <input type="number" v-model="slideBtnWidth" class="icon_input" style="width: 33%" />
            <input type="number" v-model="slideBtnHeight" class="icon_input" style="width: 33%" />
          </div>
        </div>
        
        <div class="labeltype">
          <div class="label">Icon Position</div>
          <div>
            <!-- <input type="text" v-model="IconPosition" class="icon_input"> -->
            <select v-model="IconPosition" class="icon_input">
              <option value="top">Top</option>
              <option value="center">Center</option>
              <option value="bottom">Bottom</option>
            </select>
          </div>
        </div>
      </div>

      <!-- Right side -->
      <div>
        <div ref="toolbox" style="display: flex; justify-content: space-around; border: 1px solid; padding: 10px">
          <template v-for="(element, index) in draggableElements" :key="index">
            <div class="draggable" :data-element-type="element.type" :draggable="true" @dragstart="handleDragStart($event, element)" style="display: inline-block; width: 20px; cursor: grab">
              <i :class="element.label"> {{ element.number }}</i>
            </div>
          </template>
        </div>
        <form @submit.prevent="addSlide">
          <div class="labeltype">
            <div class="label">Image</div>
            <span style="display: flex; column-gap: 2px">
              <label class="custom-file-input">
                <i class="fa-solid fa-file"></i>
                <input type="file" id="handleImageIcon" @change="handleImage" />
              </label>
              <button type="submit" class="topBarButtons" style="color: #fff">Add Slide</button>
            </span>
          </div>
        </form>
        <button @click="handleCodeModel" class="button codebutton"><i class="fa fa-code" aria-hidden="true"></i></button>
      </div>
    </div>

    <span ref="slider_container" @dragstart="handleDrag2($event)" style="display: block; position: relative">
      <div class="slider" ref="slideBox" style="position: relative; width: 900px; height: 500px; overflow: hidden; margin: 50px; border: 1px dashed gray" data-slide="sliding">
        <div :class="sliderName" :style="{ display: 'flex', transition: 'transform 0.5s ease-in-out', transform: `translateX(-${currentSlideIndex * slideWidth}px)` }" ref="slides">
          <div :class="slideImg" v-for="(slide, index) in slides" :key="index" ref="slide" :style="{ backgroundColor: 'aqua', flex: '0 0 auto', width: `${sliderWidth}`, height: `${sliderHeight}`, position: 'relative' }" @dragover.prevent="handleDragOver" @drop="handleDrop">
            <img :src="slide" style="width: 100%; height: 100%" />
          </div>
        </div>
        <div @click="prevSlide" :class="sliderPreviosBtn" :style="{ position: 'absolute', top: getTopPosition(), minWidth: `${slideBtnWidth}px`, height: `${slideBtnHeight}px`, border: '1px dashed gray' }">
          <div v-if="Previousicon" :style="{ background: `url(${Previousicon})`, backgroundSize: 'cover', width: `${slideBtnWidth}px`, height: `${slideBtnHeight}px`, margin: `${marginY}px ${marginX}px` }"></div>
          <div v-else :style="{ minWidth: `${slideBtnWidth}px`, height: `${slideBtnHeight}px`, margin: `${marginY}px ${marginX}px` }">{{ Previousicontext }}</div>
        </div>
        <div @click="nextSlide" :class="sliderNextBtn" :style="{ position: 'absolute', top: getTopPosition(), right: '0', minWidth: `${slideBtnWidth}px`, height: `${slideBtnHeight}px`, border: '1px dashed gray' }">
          <div v-if="Nexticon" :style="{ background: `url(${Nexticon})`, backgroundSize: 'cover', width: `${slideBtnWidth}px`, height: `${slideBtnHeight}px`, margin: `${marginY}px ${marginX}px` }"></div>
          <div v-else :style="{ minWidth: `${slideBtnWidth}px`, height: `${slideBtnHeight}px`, margin: `${marginY}px ${marginX}px` }">{{ Nexticontext }}</div>
        </div>
      </div>
    </span>
  </div>
  <!-- <div @dragover.prevent="handleDragOver2" @drop="handleDrop2" class="span"></div> -->

  <code class="showcode custom-scroll" ref="codemodel">
    <button @click="copy" class="copy"><i class="fa-solid fa-copy"></i></button>{{ code }}
  </code>
</template>

<script>
import '@fortawesome/fontawesome-free/css/all.css'; // Import FontAwesome CSS

  export default {
    data() {
      return {
        html: "",
        backgroundcolor: "#ffffff",
        currentSlideIndex: 0,
        slideWidth: 500,
        sliderWidth: "900",
        sliderHeight: "500",
        slides: [],
        newSlide: "",
        Previousicontext: "",
        Previousicon: "",
        Nexticon: "",
        Nexticontext: "",
        code: "",
        dropslide: "",
        marginX: "0",
        marginY: "0",
        IconPosition: "center",
        draggableElements: [
          { type: "div", label: "fa-solid fa-square" },
          { type: "h1", label: "fa-solid fa-h", number: "1" },
          { type: "h2", label: "fa-solid fa-h", number: "2" },
          { type: "h3", label: "fa-solid fa-h", number: "3" },
          { type: "h4", label: "fa-solid fa-h", number: "4" },
          { type: "h5", label: "fa-solid fa-h", number: "5" },
          { type: "h6", label: "fa-solid fa-h", number: "6" },
          // { type: 'img', label: 'Img' ,number:'6'},
        ],
        selectedElement: null,
        slideBtnWidth: "25",
        slideBtnHeight: "25",
        sliderName: "Slider" + this.random(),
        sliderPreviosBtn: "Previous" + this.random(),
        sliderNextBtn: "Next" + this.random(),
        slideImg: "slide" + this.random(),
        dropslideBox: 2345,
      };
    },
    computed: {
      getImageUrl() {
        return (iconName) => {
          if (!iconName) return "";
          return require("@/assets/slideImage/" + iconName);
        };
      },
    },

    methods: {
      random() {
        return Math.floor(Math.random() * 1000);
      }, 
      handleDrag2(event) {
        let a = event.target.getAttribute("data-slide");
        console.log(a);
      },
      // Drag over event handler
      handleDragOver2(event) {
        event.preventDefault();
      },
      handleDrop2(event) {
        event.preventDefault();
        const elementType = event.dataTransfer.getData("text/html");
        const slide = event.target.closest(".span");
        console.log(slide);
        if (slide) {
          const newElement = document.createElement("span");
          newElement.classList.add(`box${this.dropslideBox}`);
          this.genrateCode();
          newElement.innerHTML = this.dropslide; // Append the new element to the slide
          slide.appendChild(newElement);

          console.log(newElement, "newElement");
          let sliderJs = "setTimeout(() => { \n alert('fr') \n let box =document.querySelector('.box" + this.dropslideBox + "');console.log(box,'box');\n   const sliderContainer = box.querySelector('." + this.sliderName + "');console.log(sliderContainer,'sliderContainer') \n  const slides = box.querySelectorAll('." + this.slideImg + "'); console.log(slides)\n   const slideWidth = slides[0].offsetWidth \n console.log(slideWidth) \n let currentSlideIndex = 1;\n    function moveSlides(direction) {\n  if (direction === '" + this.sliderPreviosBtn + "') {\n   currentSlideIndex = Math.max(currentSlideIndex - 1, 0);\n   } else {\n  currentSlideIndex = Math.min(currentSlideIndex + 1, slides.length - 1);\n   }\n sliderContainer.style.transform = `translateX(-${currentSlideIndex * slideWidth}px)`;\n }\n const previousBtn = box.querySelector('." + this.sliderPreviosBtn + "'); console.log(previousBtn,'prevBtn')\n   previousBtn.addEventListener('click', function() {\n    moveSlides('" + this.sliderPreviosBtn + "');\n   });\n   const nextBtn = box.querySelector('." + this.sliderNextBtn + "');console.log(nextBtn,'nextbtn')\n nextBtn.addEventListener('click', function() { \n   moveSlides('" + this.sliderNextBtn + "');\n   });\n  },1000) ";
          const scriptTag = document.createElement("script");
          scriptTag.innerHTML = sliderJs;
          newElement.appendChild(scriptTag);
          this.dropslideBox++;
        }
      },
      handleCodeModel() {
        const model = this.$refs.codemodel;
        this.genrateCode();
        model.classList.toggle("open-modal");
      },
      async copy() {
        let text = document.querySelector(".showcode").textContent;
        try {
          await navigator.clipboard.writeText(text);
          alert("Content copied to clipboard");
          return true; // Indicate success
        } catch (err) {
          console.error("Failed to copy: ", err);
          return false; // Indicate failure
        }
      },

      handleDragStart(event, element) {
        console.log(event.target.dataset.elementType);
        event.dataTransfer.setData("text/plain", event.target.dataset.elementType);
      },

      // Drag over event handler
      handleDragOver(event) {
        event.preventDefault();
      },

      handleDrop(event) {
        event.preventDefault();
        const elementType = event.dataTransfer.getData("text/plain");
        console.log("Dropped element type:", elementType);

        // Find the slide where the element is dropped
        const slide = event.target.closest(".slide");
        if (slide) {
          const newElement = this.createFormElement(elementType, slide, `form-element`);
        }
      },
      selectitem(event) {
        const textareaElements = event.querySelectorAll("div");

        for (let i = 0; i < textareaElements.length; i++) {
          textareaElements[i].addEventListener("click", () => {
            this.selectElement(textareaElements[i]);
          });
        }
      },
      selectElement(event) {
        this.selectedElement = event;

        console.log("Selected Element:", this.selectedElement);
      },
      getTopPosition() {
        switch (this.IconPosition) {
          case "top":
            return `0px`;
          case "center":
            return `${this.sliderHeight / 2 - this.slideBtnHeight}px`;
          case "bottom":
            return `${this.sliderHeight - this.slideBtnHeight}px`;
          default:
            return "0";
        }
      },
      openFilePrevInput() {
        document.getElementById("handlePrevIcon").click();
      },
      openFileNextInput() {
        document.getElementById("handleNextIcon").click();
      },
      openFileImageInput() {
        document.getElementById("handleImageIcon").click();
      },

      handlePrevIconChange(e) {
        this.handleIcon(e, "Previousicon");
      },
      handleNextIconChange(e) {
        this.handleIcon(e, "Nexticon");
      },
      handleIcon(e, iconProperty) {
        let inputVal = e.target;
        if (inputVal.files.length > 0) {
          var reader = new FileReader();
          reader.onloadend = () => {
            this[iconProperty] = reader.result;
          };
          reader.readAsDataURL(inputVal.files[0]);
        } else {
          this[iconProperty] = "";
        }
      },
      handleImage(e) {
        let inputVal = e.target;
        if (inputVal.files.length > 0) {
          var reader = new FileReader();
          reader.onloadend = () => {
            // this.selectedElement.firstElementChild.src = reader.result;
            this.newSlide = reader.result;
          };
          reader.readAsDataURL(inputVal.files[0]);
        } else {
          inputVal = "";
        }
      },
      addSlide() {
        this.slides.push(this.newSlide);
        this.newSlide = "";
        this.handleLayout();
      },
      prevSlide() {
        if (this.currentSlideIndex > 0) {
          this.currentSlideIndex--;
        }
      },
      nextSlide() {
        if (this.currentSlideIndex < this.slides.length - 1) {
          this.currentSlideIndex++;
        }
      },
      handleLayout() {
        const sliderBox = this.$refs.slideBox;
        const slides = this.$refs.slide;
        sliderBox.style.cssText += `width:${this.sliderWidth}px; height:${this.sliderHeight}px; background-color:${this.backgroundcolor}`;
        if (slides) {
          slides.forEach((slide) => {
            slide.style.cssText += `width:${this.sliderWidth}px;height:${this.sliderHeight}px;`;
          });
        }
        this.slideWidth = this.sliderWidth;
      },
      genrateCode() {
        let sliderBox = this.$refs.slider_container;

        // let scriptExists = sliderBox.querySelector("script");
        // if (!scriptExists) {
        //   // let sliderJs = "\n document.addEventListener('DOMContentLoaded', function() { \n alert('fr') \n   const sliderContainer = document.querySelector('" + a + "');\n  const slides = document.querySelectorAll('.slide');\n   const slideWidth = slides[0].offsetWidth;\n let currentSlideIndex = 1;\n    function moveSlides(direction) {\n  if (direction === 'previous') {\n   currentSlideIndex = Math.max(currentSlideIndex - 1, 0);\n   } else {\n  currentSlideIndex = Math.min(currentSlideIndex + 1, slides.length - 1);\n   }\n sliderContainer.style.transform = `translateX(-${currentSlideIndex * slideWidth}px)`;\n }\n const previousBtn = document.querySelector('.previousbtn');\n   previousBtn.addEventListener('click', function() {\n    moveSlides('previous');\n   });\n   const nextBtn = document.querySelector('.nextbtn');\n   nextBtn.addEventListener('click', function() { \n   moveSlides('next');\n   });\n   });\n   ";
        //   let sliderJs = "setTimeout(() => { \n alert('fr') \n let box =document.querySelector('.box" + this.dropslideBox + "');console.log(box,'box');\n   const sliderContainer = box.querySelector('." + this.sliderName + "');console.log(sliderContainer,'sliderContainer') \n  const slides = box.querySelectorAll('." + this.slideImg + "'); console.log(slides)\n   const slideWidth = slides[0].offsetWidth \n console.log(slideWidth) \n let currentSlideIndex = 1;\n    function moveSlides(direction) {\n  if (direction === '" + this.sliderPreviosBtn + "') {\n   currentSlideIndex = Math.max(currentSlideIndex - 1, 0);\n   } else {\n  currentSlideIndex = Math.min(currentSlideIndex + 1, slides.length - 1);\n   }\n sliderContainer.style.transform = `translateX(-${currentSlideIndex * slideWidth}px)`;\n }\n const previousBtn = box.querySelector('." + this.sliderPreviosBtn + "'); console.log(previousBtn,'prevBtn')\n   previousBtn.addEventListener('click', function() {\n    moveSlides('" + this.sliderPreviosBtn + "');\n   });\n   const nextBtn = box.querySelector('." + this.sliderNextBtn + "');console.log(nextBtn,'nextbtn')\n nextBtn.addEventListener('click', function() { \n   moveSlides('" + this.sliderNextBtn + "');\n   });\n  },1000) ";
        //   const scriptTag = document.createElement("script");
        //   scriptTag.innerHTML = sliderJs;
        //   sliderBox.appendChild(scriptTag);
        // }

        this.code = sliderBox.outerHTML;
        this.dropslide = sliderBox.outerHTML;
      },

      createFormElement(elementType, canvas, referenceId) {
        const element = canvas;
        element.classList.add("form-element");
        const randomClass = this.generateRandomClassName();
        element.classList.add(`formElement-${randomClass}`);
        element.dataset.elementType = elementType;

        switch (elementType) {
          case "h1": {
            var newDiv = document.createElement("h1");
            newDiv.style.cssText += `position:absolute;top:0%;outline-offset: 6px;outline: gray dashed 0.5px;`;
            newDiv.textContent = "Dropped element: ";
            newDiv.setAttribute("contenteditable", "true");
            newDiv.classList.add(`${this.generateRandomClassName()}`);
            this.makeDraggable(newDiv);
            element.appendChild(newDiv);
            break;
          }
          case "h2": {
            var newDiv = document.createElement("h2");
            newDiv.style.cssText += `position:absolute;top:0%;outline-offset: 6px;outline: gray dashed 0.5px;`;
            newDiv.textContent = "Dropped element: ";
            newDiv.setAttribute("contenteditable", "true");
            newDiv.classList.add(`${this.generateRandomClassName()}`);
            this.makeDraggable(newDiv);
            element.appendChild(newDiv);
            break;
          }
          case "h3": {
            var newDiv = document.createElement("h3");
            newDiv.style.cssText += `position:absolute;top:0%;outline-offset: 6px;outline: gray dashed 0.5px;`;
            newDiv.textContent = "Dropped element: ";
            newDiv.setAttribute("contenteditable", "true");
            newDiv.classList.add(`${this.generateRandomClassName()}`);
            this.makeDraggable(newDiv);
            element.appendChild(newDiv);
            break;
          }
          case "h4": {
            var newDiv = document.createElement("h4");
            newDiv.style.cssText += `position:absolute;top:0%;outline-offset: 6px;outline: gray dashed 0.5px;`;
            newDiv.textContent = "Dropped element: ";
            newDiv.setAttribute("contenteditable", "true");
            newDiv.classList.add(`${this.generateRandomClassName()}`);
            this.makeDraggable(newDiv);
            element.appendChild(newDiv);
            break;
          }
          case "h5": {
            var newDiv = document.createElement("h5");
            newDiv.style.cssText += `position:absolute;top:0%;outline-offset: 6px;outline: gray dashed 0.5px;`;
            newDiv.textContent = "Dropped element: ";
            newDiv.setAttribute("contenteditable", "true");
            newDiv.classList.add(`${this.generateRandomClassName()}`);
            this.makeDraggable(newDiv);
            element.appendChild(newDiv);
            break;
          }
          case "h6": {
            var newDiv = document.createElement("h6");
            newDiv.style.cssText += `position:absolute;top:0%;outline-offset: 6px;outline: gray dashed 0.5px;`;
            newDiv.textContent = "Dropped element: ";
            newDiv.setAttribute("contenteditable", "true");
            newDiv.classList.add(`${this.generateRandomClassName()}`);
            this.makeDraggable(newDiv);
            element.appendChild(newDiv);
            break;
          }
          case "div": {
            var newDiv = document.createElement("div");
            newDiv.style.cssText += `position:absolute;top:0%;outline-offset: 6px;outline: gray dashed 0.5px;`;
            newDiv.textContent = "Dropped element: ";
            newDiv.setAttribute("contenteditable", "true");
            newDiv.classList.add(`${this.generateRandomClassName()}`);
            this.makeDraggable(newDiv);

            element.appendChild(newDiv);
            break;
          }

          // Add similar cases for other element types
        }
        this.selectitem(element);

        return element;
      },

      generateRandomClassName() {
        return "div-" + Math.floor(Math.random() * 1000);
      },
      makeDraggable(element) {
        let offsetX,
          offsetY,
          isDraggingElement = false;

        // Mouse down event handler
        element.addEventListener("mousedown", (e) => {
          isDraggingElement = true;

          // Get the initial mouse cursor position
          offsetX = e.clientX - element.getBoundingClientRect().left;
          offsetY = e.clientY - element.getBoundingClientRect().top;

          // Set the cursor style to "grabbing" during drag
          element.style.cursor = "grabbing";
        });

        // Mouse move event handler
        document.addEventListener("mousemove", (e) => {
          if (isDraggingElement) {
            // Calculate the new position based on mouse movement
            const parentRect = element.parentElement.getBoundingClientRect();
            const newX = ((e.clientX - offsetX - parentRect.left) / parentRect.width) * 100;
            const newY = ((e.clientY - offsetY - parentRect.top) / parentRect.height) * 100;

            // Check if the new position is within the parent boundaries
            const maxX = 100 - (element.clientWidth / parentRect.width) * 100;
            const maxY = 100 - (element.clientHeight / parentRect.height) * 100;
            // Set the new position within the parent boundaries
            element.style.left = `${Math.min(Math.max(newX, 0), maxX)}%`;
            element.style.top = `${Math.min(Math.max(newY, 0), maxY)}%`;
          }
        });

        // Mouse up event handler
        document.addEventListener("mouseup", () => {
          isDraggingElement = false;
          // Set the cursor style back to "grab" after drag
          element.style.cursor = "grab";
        });
      },
    },
  };
</script>

<style scoped>
.codebutton{
  position: absolute;
    top: 0;
    width: 30px !important;
    right: 10px;
}
  .span {
    width: auto;
    min-height: 500px;
    background: gray;
  }
  .custom-scroll::-webkit-scrollbar-track {
    background: #f1f1f1;
    /* Light grey background */
    border-radius: 10px;
    /* Optional: Rounds the corners of the track */
  }

  /* This styles the scrollbar thumb (the part that's draggable) */
  .custom-scroll::-webkit-scrollbar-thumb {
    background: #888;
    /* Darker grey scrollbar thumb */
    border-radius: 10px;
    /* Optional: Rounds the corners of the thumb */
  }

  /* This changes the scrollbar thumb color on hover */
  .custom-scroll::-webkit-scrollbar-thumb:hover {
    background: #555;
    /* Even darker grey on hover */
  }

  /* This styles the scrollbar itself (including the thumb, the track, and the buttons, but in this case, we're just setting its width) */
  .custom-scroll::-webkit-scrollbar {
    width: 2px;
    /* Width of the scrollbar */
    height: 0px;
  }

  .open-modal {
    transform: translateX(0%) !important;
  }

  .showcode {
    width: 286px;
    word-wrap: break-word;
    display: inline-block;
    overflow-y: auto;
    padding: 20px;
    text-align: justify;
    background: #333333;
    color: #fff;
    height: 97vh;
    position: fixed;
    top: 0px;
    right: 0;
    transition: 0.5s all ease;
    transform: translateX(100%);
  }

  .copy {
    position: absolute;
    top: 0;
    line-height: 14px;
    padding: 5px;
    box-sizing: border-box;
    right: 0;
    cursor: pointer;
  }

  .topBarButtons {
    cursor: pointer;
    padding: 6px 17px;
    /* padding-top: 0px; */
    border-radius: 5px;
    background: #4caf50;
    transition: 0.5s;
    border: 0.5px solid lightgray;
    margin-right: 5px;
  }

  .pull-left {
    padding: 6px 10px;
    border-radius: 5px;
    background: white;
    transition: 0.5s;
    border: 0.5px solid lightgray;
    position: relative;
  }

  /* Container styles */
  .container {
    width: 275px;
    /* margin-top: 40px; */
    background-color: #333333;
    color: #fff;
    padding: 8px;
    font-size: 0.8rem;
    height:97.3vh;
    position: relative;
  }

  .labelheading {
    display: block;
    font-size: 1em;
    padding: 0.5em 0.5em 0.5em 0.7em;
  }

  .labeltype {
    height: 100%;
    line-height: 22px;
    display: grid;
    grid-template-columns: 30% 70%;
    column-gap: 23%;
    margin-bottom: 0.7em;
  }

  .label {
    display: block !important;
    margin-bottom: 3px !important;
    margin-top: 2px !important;
    white-space: nowrap !important;
  }

  .icon_input {
    line-height: 14px;
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
    box-sizing: border-box;
    width: 47%;
  }

  /* Right side styles */
  .right-side2 {
    width: 45%;
  }

  /* Form styles */
  form {
    margin-top: 20px;
  }

  .button {
    line-height: 14px;
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
    box-sizing: border-box;
    width: 100%;
    margin-bottom: 10px;
    background-color: #4caf50;
    color: white;
    cursor: pointer;
  }

  .button:hover {
    background-color: #45a049;
  }

  .custom-file-input {
    width: 8%;
    position: relative;
    overflow: hidden;
    display: inline-block;
    cursor: pointer;
    background-color: #fff;
    color: #000;
    padding: 2px 4px;
    border-radius: 5px;
    border: 0.5px solid lightgray;
  }

  .custom-file-input input[type="file"] {
    font-size: 0px;
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    opacity: 0;
    cursor: pointer;
  }
</style>
