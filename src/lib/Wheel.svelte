<script>
  import { tweened } from 'svelte/motion';
  import { cubicOut } from 'svelte/easing';

  const initialItems = [
    "Where's the peg?",
    "Ring of fire",
    "Dice",
    "Soldier",
    "To solder",
    "Arm wrestle bouncer",
    "Hi everybody",
    "Tied shoe laces",
    "Need a hand",
    "Dog",
    "Leash",
    "Lock of hair",
    "Yes man",
    "Spank stick"
  ];

  let items = initialItems.map(item => ({ name: item, crossed: false }));

  const colors = [
    '#FFC0CB', '#FFD700', '#98FB98', '#87CEFA', '#DDA0DD', '#F0E68C',
    '#FF6347', '#E6E6FA', '#20B2AA', '#FFA07A', '#FF69B4', '#CD853F',
    '#BA55D3', '#FF8C00'
  ];

  const radius = 200;
  const centerX = 250;
  const centerY = 250;
  const angleStep = (2 * Math.PI) / items.length;

  let rotation = tweened(0, {
    duration: 5000,
    easing: cubicOut
  });

  let isSpinning = false;
  let result = '';
  let totalRotation = 0;

  async function spin() {
    if (isSpinning) return;
    
    isSpinning = true;
    result = '';
    
    let validResult = false;
    while (!validResult) {
      const spinAmount = 1440 + Math.random() * 360; // At least 4 full rotations + random
      totalRotation += spinAmount;
      
      await rotation.set(totalRotation);
      
      const finalRotation = totalRotation % 360;
      const resultIndex = items.length - 1 - Math.floor(finalRotation / (360 / items.length));
      
      if (!items[resultIndex].crossed) {
        items[resultIndex].crossed = true;
        result = items[resultIndex].name;
        validResult = true;
      }
    }
    
    isSpinning = false;
    items = items; // Trigger reactivity
  }

  function resetWheel() {
    items = initialItems.map(item => ({ name: item, crossed: false }));
    result = '';
    totalRotation = 0;
    rotation.set(0);
  }
</script>

<div class="wheel-container">
  <svg width="500" height="500" viewBox="0 0 500 500">
    <defs>
      <clipPath id="wheelClip">
        <circle cx={centerX} cy={centerY} r={radius} />
      </clipPath>
    </defs>
    
    <g clip-path="url(#wheelClip)">
      <g transform={`rotate(${$rotation}, ${centerX}, ${centerY})`}>
        <!-- Item slices -->
        {#each items as item, i}
          {@const startAngle = i * angleStep - Math.PI / 2}
          {@const endAngle = (i + 1) * angleStep - Math.PI / 2}
          {@const x1 = centerX + radius * Math.cos(startAngle)}
          {@const y1 = centerY + radius * Math.sin(startAngle)}
          {@const x2 = centerX + radius * Math.cos(endAngle)}
          {@const y2 = centerY + radius * Math.sin(endAngle)}
          
          <path
            d={`M ${centerX},${centerY} L ${x1},${y1} A ${radius},${radius} 0 0,1 ${x2},${y2} Z`}
            fill={colors[i]}
            stroke="white"
            stroke-width="2"
          />
          
          <!-- Item text -->
          {@const textAngle = (startAngle + endAngle) / 2}
          {@const textRadius = radius * 0.75}
          {@const textX = centerX + textRadius * Math.cos(textAngle)}
          {@const textY = centerY + textRadius * Math.sin(textAngle)}
          
          <text
            x={textX}
            y={textY}
            text-anchor="middle"
            dominant-baseline="middle"
            transform={`
              rotate(${(textAngle * 180) / Math.PI + 90}, ${textX}, ${textY})
              rotate(${-90}, ${textX}, ${textY})
            `}
            font-size="10"
            fill="black"
            text-decoration={item.crossed ? "line-through" : "none"}
          >
            {item.name}
          </text>
        {/each}
      </g>
    </g>

    <!-- Center image -->
    <image
      href="https://i.ibb.co/6HrqsxT/image.png"
      x={centerX - 50}
      y={centerY - 50}
      width="100"
      height="100"
    />

    <!-- Pointer -->
    <path d="M 250 50 L 260 10 L 240 10 Z" fill="red" />
  </svg>

  <button on:click={spin} disabled={isSpinning}>
    {isSpinning ? 'Spinning...' : 'Spin the Wheel'}
  </button>

  {#if result}
    <p>Result: {result}</p>
  {/if}

  <button on:click={resetWheel}>Reset Wheel</button>
</div>

<style>
  .wheel-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 20px;
  }

  button {
    font-size: 18px;
    padding: 10px 20px;
    cursor: pointer;
  }

  button:disabled {
    cursor: not-allowed;
  }

  p {
    font-size: 24px;
    font-weight: bold;
  }
</style>
