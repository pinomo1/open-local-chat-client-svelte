<script>
    let buttonProps = {
      class: '', // Initialize the class property as an empty string
      on: {
        click: undefined,
        mouseover: undefined,
        mouseenter: undefined,
        mouseleave: undefined,
      },
      text: 'undefined',

      ...$$restProps, // Spread the rest of the props

      
    };
  </script>
  
  <button {...buttonProps}>
    {buttonProps.text}
    <slot />
  </button>
  
  <style>
    /* * {
      width: 5rem;
      height: 1.5rem;
      border: 1px solid black;
      padding: 0.5em;
      border-radius: 1em;
      background-color: white;
      cursor: pointer;
    }
     */

    .primary {
      color: green;
    }
    .primary:hover {
      background-color: green;
      color: white;
    }
    .danger {
      color: red;
    }
    .sm {
      font-size: 0.9em;
      font: 700;
      padding: 0.1em;
      width: 5rem;
      height: 2rem;
      border: 1px solid black;
      padding: 0.5em;
      border-radius: 1em;
      background-color: white;
      cursor: pointer;
    }
    .lg {
        font-size: 1.5em;
        padding: 0.5em;
        width: 5rem;
        height: 2rem;
        border: 1px solid black;
        padding: 0.5em;
        border-radius: 1em;
        background-color: white;
        cursor: pointer;
    }
  </style>