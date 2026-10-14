<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Sorteo Aleatorio</title>
  <link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700;900&family=DM+Mono:wght@400;500&display=swap" rel="stylesheet"/>
  <style>
    :root {
      --bg: #0d0d0d;
      --card: #161616;
      --border: #2a2a2a;
      --accent: #e8c547;
      --accent2: #e87c47;
      --text: #f0ede6;
      --muted: #ffffff;
      --radius: 4px;
    }

    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: 'DM Mono', monospace;
      background: var(--bg);
      color: var(--text);
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      padding: 2rem;
      overflow-x: hidden;
    }

    /* Grain overlay */
    body::before {
      content: '';
      position: fixed;
      inset: 0;
      background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)' opacity='0.04'/%3E%3C/svg%3E");
      pointer-events: none;
      z-index: 999;
      opacity: 0.4;
    }

    .container {
      width: 100%;
      max-width: 560px;
      display: flex;
      flex-direction: column;
      gap: 2rem;
    }

    header {
      text-align: center;
    }

    .tag {
      font-size: 0.85rem;
      letter-spacing: 0.25em;
      text-transform: uppercase;
      color: var(--accent);
      border: 1px solid var(--accent);
      display: inline-block;
      padding: 0.25rem 0.75rem;
      border-radius: 2px;
      margin-bottom: 1rem;
    }

    h1 {
      font-family: 'Playfair Display', serif;
      font-size: clamp(2.2rem, 6vw, 3.5rem);
      font-weight: 900;
      line-height: 1;
      letter-spacing: -0.02em;
      color: var(--text);
    }

    h1 span {
      color: var(--accent);
    }

    /* Result display */
    .result-box {
      background: var(--card);
      border: 1px solid var(--border);
      border-radius: var(--radius);
      padding: 2.5rem 2rem;
      text-align: center;
      position: relative;
      overflow: hidden;
      min-height: 130px;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      gap: 0.5rem;
    }

    .result-box::before {
      content: '';
      position: absolute;
      top: 0; left: 0; right: 0;
      height: 2px;
      background: linear-gradient(90deg, var(--accent2), var(--accent));
      transform: scaleX(0);
      transform-origin: left;
      transition: transform 0.4s ease;
    }

    .result-box.revealed::before {
      transform: scaleX(1);
    }

    .result-label {
      font-size: 0.6rem;
      letter-spacing: 0.2em;
      text-transform: uppercase;
      color: var(--muted);
    }

    .result-value {
      font-family: 'Playfair Display', serif;
      font-size: clamp(1.6rem, 5vw, 2.4rem);
      font-weight: 700;
      color: var(--text);
      line-height: 1.2;
      opacity: 0;
      transform: translateY(10px);
      transition: opacity 0.35s ease, transform 0.35s ease;
    }

    .result-value.show {
      opacity: 1;
      transform: translateY(0);
    }

    .result-value.idle {
      font-size: 1rem;
      font-family: 'DM Mono', monospace;
      color: var(--muted);
      font-weight: 400;
      opacity: 1;
      transform: none;
    }

    .result-index {
      font-size: 0.65rem;
      color: var(--muted);
      letter-spacing: 0.1em;
      opacity: 0;
      transition: opacity 0.35s 0.2s ease;
    }

    .result-index.show {
      opacity: 1;
    }

    /* Selector */
    .select-wrap {
      display: flex;
      flex-direction: column;
      gap: 0.4rem;
    }

    .select-label {
      font-size: 0.6rem;
      letter-spacing: 0.2em;
      text-transform: uppercase;
      color: var(--muted);
    }

    .select {
      width: 100%;
      padding: 0.85rem 1rem;
      font-family: 'DM Mono', monospace;
      font-size: 0.85rem;
      letter-spacing: 0.05em;
      background: var(--card);
      color: var(--text);
      border: 1px solid var(--border);
      border-radius: var(--radius);
      cursor: pointer;
      appearance: none;
      background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='8' viewBox='0 0 12 8'%3E%3Cpath d='M1 1l5 5 5-5' stroke='%23e8c547' stroke-width='2' fill='none' fill-rule='evenodd'/%3E%3C/svg%3E");
      background-repeat: no-repeat;
      background-position: right 1rem center;
      transition: border-color 0.15s ease;
    }

    .select:hover,
    .select:focus {
      border-color: var(--accent);
      outline: none;
    }

    /* Button */
    .btn-wrap {
      position: relative;
    }

    .btn {
      width: 100%;
      padding: 1rem 2rem;
      font-family: 'DM Mono', monospace;
      font-size: 0.85rem;
      letter-spacing: 0.12em;
      text-transform: uppercase;
      background: var(--accent);
      color: #0d0d0d;
      border: none;
      border-radius: var(--radius);
      cursor: pointer;
      font-weight: 500;
      position: relative;
      overflow: hidden;
      transition: transform 0.15s ease, box-shadow 0.15s ease;
    }

    .btn::after {
      content: '';
      position: absolute;
      inset: 0;
      background: rgba(255,255,255,0.15);
      transform: translateX(-100%);
      transition: transform 0.3s ease;
    }

    .btn:hover::after {
      transform: translateX(0);
    }

    .btn:active {
      transform: scale(0.98);
    }

    .btn:hover {
      box-shadow: 0 0 24px rgba(232, 197, 71, 0.3);
    }

    .btn.spinning {
      pointer-events: none;
      opacity: 0.7;
    }


    footer {
      text-align: center;
      font-size: 0.6rem;
      color: var(--muted);
      letter-spacing: 0.1em;
    }
  </style>
</head>
<body>

<div class="container">

  <header>
    <div class="tag">Temas de la Jam Poblet</div>
    <h1>Elige <span>al azar</span></h1>
  </header>

  <!-- Resultado -->
  <div class="result-box" id="resultBox">
    <div class="result-label" id="resultLabel">Pulsa el botón para elegir tema</div>
    <div class="result-value idle" id="resultValue">— — —</div>
    <div class="result-index" id="resultIndex"></div>
  </div>

  <!-- Selector de filtro -->
  <div class="select-wrap">
    <label for="filterSelect" class="select-label">Filtrar temas</label>
    <select id="filterSelect" class="select">
      <option value="todos">Todos los temas</option>
      <option value="una">Sólo temas de la jam (*)</option>
      <option value="dos">Sólo temas de especial interés (#)</option>
    </select>
  </div>

  <!-- Botón -->
  <div class="btn-wrap">
    <button class="btn" id="drawBtn" onclick="draw()">
      ✦ &nbsp; Pulsa
    </button>
  </div>

  <footer>Resultado generado de forma totalmente aleatoria</footer>

</div>

<script>
  // ──────────────────────────────────────────────
  //  ✏️  EDITA AQUÍ TU LISTA DE VALORES
  // Si tiene un "*" es que es de la jam. Si tiene un "#" es que es un tema "especial", interesante
  // ──────────────────────────────────────────────
  const valores = [
"*#All Blues",
"*All of me",
"*#Autum Leaves",
"*Blue Bossa",
"*Blue in Green",
"*Blue Monk",
"*Blue Trane",
"Cantaloupe Island",
"*(The) Days of Wine and Roses",
"*Equinox",
"FootPrints",
"*Freddie The Freeloader",
"* (The) Girl from Ipanema",
"*How Insensitive",
"I'm beginning to see the light",
"*In a Sentimental Mood",
"*I Wish I Knew",
"*It don't mean a thing",
"*Killer Joe",
"Lady Bird",
"*Little Sunflower",
"*Manha de Carnaval",
"(This) Masquerade",
"*Mercy, Mercy, Mercy",
"*Mr. P.C.",
"#Moanin'",
"*My Funny Valentine",
"My Little Suede Shoes",
"*Night And Day",
"Nostalgia in Times Square",
"Route 66",
"Satin Doll",
"*SideWinder",
"Solar",
"*Someday my Prince will come",
"*Song for my Father",
"*So What",
"*St. Thomas",
"Stella by Starlight",
"*Stolen Moments",
"*Stomping at the Savoy",
"*Take Five",
"*Take the 'A' train",
"Tenor Madness",
"Well you needn't",
"*Work Song",
  ];
  // ──────────────────────────────────────────────

  let lastIndex = -1;

  const resultBox    = document.getElementById('resultBox');
  const resultLabel  = document.getElementById('resultLabel');
  const resultValue  = document.getElementById('resultValue');
  const resultIndex  = document.getElementById('resultIndex');
  const drawBtn      = document.getElementById('drawBtn');
  const filterSelect = document.getElementById('filterSelect');

  // Devuelve true/false según si el valor cumple el filtro elegido
  function cumpleFiltro(valor, filtro) {
    switch (filtro) {
      case 'dos':   return valor.includes('#');
      case 'una':   return valor.includes('*');
      case 'todos':
      default:      return true;
    }
  }

  function draw() {
    if (valores.length === 0) return;

    const filtro = filterSelect.value;

    // Construir la lista de candidatos según el filtro,
    // conservando su índice original en "valores"
    const candidatos = valores
      .map((valor, idx) => ({ valor, idx }))
      .filter(item => cumpleFiltro(item.valor, filtro));

    if (candidatos.length === 0) {
      resultLabel.textContent = 'No hay temas para ese filtro';
      resultValue.textContent = '— — —';
      resultValue.classList.add('idle');
      resultIndex.classList.remove('show');
      return;
    }

    // Elegir un candidato al azar (evitando repetir el último resultado)
    let elegido;
    do {
      elegido = candidatos[Math.floor(Math.random() * candidatos.length)];
    } while (candidatos.length > 1 && elegido.idx === lastIndex);

    const finalIdx = elegido.idx;
    lastIndex = finalIdx;

	// Reset animation classes
	resultBox.classList.remove('revealed');
	resultValue.classList.remove('show', 'idle');
	resultIndex.classList.remove('show');

	// Show result with a brief fade-in
	setTimeout(() => {
	  resultValue.textContent = valores[finalIdx];
	  resultLabel.textContent = '¡Resultado!';
	  const posEnFiltro = candidatos.findIndex(c => c.idx === finalIdx) + 1;
	  resultIndex.textContent = `Elemento ${posEnFiltro} de ${candidatos.length}`;
	  resultValue.classList.add('show');
	  resultIndex.classList.add('show');
	  resultBox.classList.add('revealed');
	  drawBtn.textContent = '✦   Elegir otro tema';
	}, 80);
  }

</script>
</body>
</html>
