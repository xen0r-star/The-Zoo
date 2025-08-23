<a id="readme-top"></a>

<h1 align="center">🐾 The Zoo - Chameleon 🐾</h1>

<p align="center">
  <img src="https://img.shields.io/badge/SvelteKit-FF3E00?style=flat&logo=svelte&logoColor=white" alt="SvelteKit"/>
  <img src="https://hackatime-badge.hackclub.com/U096DLZH6KH/The%20Zoo" alt="HackaTime Badge"/>
  <img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="License"/>
  <img src="https://img.shields.io/badge/status-Active-brightgreen" alt="Status"/>
</p>

<p align="center">
  <img src="./docs/card.svg" alt="Chameleon" width="200" style="background-color: #f9f9f913; border-radius: 10px;"/>
</p>

**The Zoo** is an interactive webpage about the chameleon (the best animal on earth), built with SvelteKit for the YSWS [Hack Club Zoo](https://zoo.hackclub.com/). It covers information about it and includes an interactive page where you can play with it at the end of the reading.

<!-- <h3 align="left" style="display: flex; align-items: center; justify-content: center; gap: 7px;">
  Currently: 0 <img src="./docs/mynt.png" alt="Chameleon" width="18"/>
</h3> -->

<div align="center">
  <a href="https://xen0r-star.github.io/The-Zoo/">
    <h3>
      👉 Live Demo 👈
    </h3>
  </a>
</div>

<!-- TABLE OF CONTENTS -->
<details>
  <summary>🗂️ Table of Contents</summary>
  <ol>
    <li>
      <a href="#📌-requirements">📌 Requirements</a>
    </li>
    <li>
      <a href="#✨-information-about-the-chameleon">✨ Information about the chameleon</a>
    </li>
    <li>
      <a href="#🦎-interactive-elements">🦎 Interactive elements</a>
    </li>
    <li>
      <a href="#🎨-figma-design">🎨 Figma Design</a>
    </li>
    <li>
      <a href="#📸-screenshots">📸 Screenshots</a>
    </li>
    <li>
      <a href="#🛠️-technologies">🛠️ Technologies</a>
    </li>
    <li>
      <a href="#🥚-easter-eggs">🥚 Easter Eggs</a>
    </li>
    <li>
      <a href="#🏆-credits">🏆 Credits</a>
    </li>
  </ol>
</details>

---

## 📌 Requirements
- **Main requirements**:
  - [X] 8+ hours of coding
  - [x] Site coded with SvelteKit and statically hosted on GitHub Pages
  - [x] Theme around the best animal (the chameleon of course)
  - [x] Include the Zoo Banner component everywhere
  - [x] Include the README.md file with a description
  - [x] Information panel [5/3]
  - [x] Information panel uses $state and $derived
  - [x] Unlock a button after reading all facts
  - [] Earn mynts
- **Bonus requirements**:
  - [X] Extra hours (8h + 27h = 35h)
  - [X] Use of svelte:head in +layout.svelte
  - [X] Animal conservation status according to IUCN
  - [X] The facts panel is fully dynamic
  - [X] Use of $effect on the main page and the interactive page
  - [X] Presence of easter eggs (3/3)
  - [X] Site 100% responsive ON ALL DEVICES (except Apple Watch)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

## ✨ Information about the chameleon
### Main page
On the main page, the user takes a short journey through the trees, discovers several sections with detailed information (basics, camouflage and tongue, endangered status), checks an interactive information panel offering fun and educational facts about the chameleon, and can unlock the “Meet Chameleon” button after reading all the facts.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

## 🦎 Interactive elements
### Interactive page
My interactive page is a simple game where the user must click on flies to score points before the chameleon catches them, just like a real chameleon would do to feed.

#### External stimulus
- **Clicks on the chameleon**: Clicking the chameleon triggers a color change—be careful not to click too many times.
- **Mouse movements**: The chameleon’s eye follows the user’s cursor.

#### Internal stimulus
- **Time**: The sky color changes dynamically according to the real time.
- **Flies**: It automatically catches the flies.
- **Randomness**: The chameleon sometimes makes a mistake and attacks the mouse cursor instead of the fly.
- **Daytime life**: The chameleon has a higher chance of hitting the fly from 8 a.m. to 8 p.m.
- **Speech**: The chameleon makes random comments depending on the situation (win/lose/pause).

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

## 🎨 Figma Design

The design of this project was created in Figma.  
👉 [See the Figma here](https://www.figma.com/design/uUUiihiYirgn1eLlLTaem4/The-zoo?m=auto&t=nZ8xwFVnpctgNBH8-1)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

## 📸 Screenshots
<table>
  <tr>
    <td>
      <img src="./docs//xen0r-star.github.io_The-Zoo_.png" width="400"/>
    </td>
    <td>
      <img src="./docs/xen0r-star.github.io_The-Zoo_ (2).png" width="400"/>
    </td>
    <td>
      <img src="./docs/xen0r-star.github.io_The-Zoo_ (3).png" width="400"/>
    </td>
  </tr>
  <tr>
    <td>
      <img src="./docs/xen0r-star.github.io_The-Zoo_ (4).png" width="400"/>
    </td>
    <td>
      <img src="./docs/xen0r-star.github.io_The-Zoo_ (5).png" width="400"/>
    </td>
    <td>
      <img src="./docs/xen0r-star.github.io_The-Zoo_ (6).png" width="400"/>
    </td>
  </tr>
  <tr>
    <td>
      <img src="./docs/xen0r-star.github.io_The-Zoo_chameleon.png" width="400"/>
    </td>
    <td>
      <img src="./docs/xen0r-star.github.io_The-Zoo_chameleon (7).png" width="400"/>
    </td>
    <td>
      <img src="./docs/xen0r-star.github.io_The-Zoo_chameleon (9).png" width="400"/>
    </td>
  </tr>
</table>

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

## 🥚 Easter Eggs
- 💚 Clicking the green chameleon in the tree will trigger a cute animation for his sweetheart in the tree opposite
- 🏪 A nice ASCII art hidden in the webpage’s source code
- 🖌️ The barcode leads to the best GitHub account

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

## 🛠️ Technologies
- [![Svelte][Svelte.dev]][Svelte-url] 
- [![TypeScript][TypeScript.dev]][TypeScript-url] 
- [![Vite][Vite.dev]][Vite-url]

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

## 🏆 Credits
- Designed by: [[xen0r-star]](https://github.com/xen0r-star)
- Coded by: [[xen0r-star]](https://github.com/xen0r-star)
- Drawings by: [[xen0r-star]](https://github.com/xen0r-star)
- Original ideas: [[xen0r-star]](https://github.com/xen0r-star)
- Testing and debugging: [[xen0r-star]](https://github.com/xen0r-star)
- Documentation: [[xen0r-star]](https://github.com/xen0r-star)
- Chameleon animation: [[xen0r-star]](https://github.com/xen0r-star)
- UI/UX: [[xen0r-star]](https://github.com/xen0r-star)
- Project management: [[xen0r-star]](https://github.com/xen0r-star)
- Motivation and coffee: [[xen0r-star]](https://github.com/xen0r-star)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

<p align="center">
  <i>🖤💛❤️ Made by a Belgian 🖤💛❤️</i>
</p>




[Svelte.dev]: https://img.shields.io/badge/Svelte-4A4A55?style=for-the-badge&logo=svelte&logoColor=FF3E00
[Svelte-url]: https://kit.svelte.dev/
[TypeScript.dev]: https://img.shields.io/badge/Typescript-4A4A55?style=for-the-badge&logo=typescript&logoColor=3178C6
[TypeScript-url]: https://www.typescriptlang.org/
[Vite.dev]: https://img.shields.io/badge/Vite-4A4A55?style=for-the-badge&logo=vite&logoColor=646CFF
[Vite-url]: https://vitejs.dev/
