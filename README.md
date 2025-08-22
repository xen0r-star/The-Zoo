# The Zoo - Chameleon Interactive

This project is an interactive web page featuring a dynamic chameleon, built with SvelteKit. It was created for a contest and meets all the required guidelines.
https://zoo.hackclub.com/

## General Description

The site presents a chameleon with which the user can interact in various ways. The experience is divided into two main parts:
- A homepage with an information panel (facts) about the chameleon.
- An interactive page where the user can play against the chameleon in a mini-game.

## Information Panel

On the main page, an information panel displays several facts about the chameleon (size, diet, habitat, etc.).  
**Features:**
- The facts are shown as cards that the user can scroll through.
- After viewing all the facts, a "Meet Chameleon" button is unlocked, allowing access to the interactive page.
- This panel uses the `$state` and `$derived` runes to manage progress and unlock the button.

## Interactive Page

The interactive page offers a game where the user must catch flies faster than the chameleon.

### External Stimulus
- **Clicks and mouse movements**:  
  - The user can click on the fly to score points.
  - Moving the mouse makes the chameleon's eye follow the cursor.
  - Clicking on the chameleon changes its color (with anti-spam protection).

### Internal Stimulus
- **Time and randomness**:  
  - The chameleon attacks the fly automatically at random intervals (function `autoChameleonAttack`).
  - The sky color changes automatically depending on the time of day (variable `hour` and array `skyColor`).
  - The fly's position is randomly determined within a defined area.
  - Sometimes, the chameleon can catch the mouse instead of the fly.

## Easter eggs
- When the user clicks on the chameleon in the tree on the homepage, it makes a heart for its sweetheart in the opposite tree, and at the end of the page,