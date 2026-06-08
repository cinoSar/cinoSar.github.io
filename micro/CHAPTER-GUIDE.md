# Microeconomics Lessons — Chapter Building Guide

> **Purpose:** This document captures every convention, pattern, and content plan needed to build chapters 6–20 of the Microeconomics Lessons site. A new session should be able to read this file and produce a chapter without further clarification from the author.

---

## 1. Site Architecture

- **Static HTML** — no framework, no build step, no bundler.
- Each chapter is a single self-contained `.html` file in `micro/chapters/`.
- Shared stylesheet: `../../css/style.css` (never modify it for chapter-specific needs).
- Chapter-specific CSS goes in a `<style>` block inside the chapter's `<head>`.
- MathJax 3.2.2 (tex-svg renderer) loaded via CDN for LaTeX rendering.
- All interactivity is vanilla JavaScript in `<script>` blocks at the bottom of `<body>`.

### File naming

```
micro/chapters/ch{N}-{slug}.html
```

Examples: `ch6-firms-and-production.html`, `ch7-costs.html`.

### After creating a chapter, always:

1. **Activate the card** in `micro/index.html`: change the chapter's `<div class="chapter-card">` to `<div class="chapter-card active">`, replace `<span class="coming-soon">Coming soon</span>` with `<a href="chapters/ch{N}-{slug}.html" class="chapter-link">Start Lesson →</a>`.
2. **Update the previous chapter's forward link**: change the `<span>` in `.chapter-nav` to an `<a href="ch{N}-{slug}.html">`.
3. **Verify via bash**: check file size, balanced `<section>` tags, canvas count, link check, MathJax presence.

---

## 2. HTML Template Structure

Every chapter follows this skeleton:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chapter N: Title | Microeconomics Lessons</title>
    <link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text y='.9em' font-size='90'>📊</text></svg>">
    <link rel="stylesheet" href="../../css/style.css">
    <script>
        window.MathJax = {
            tex: { inlineMath: [['\\(', '\\)']], displayMath: [['\\[', '\\]']] },
            svg: { fontCache: 'global' }
        };
    </script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/3.2.2/es5/tex-svg.js" async></script>
    <style>
        /* COPY the chapter-specific CSS block verbatim from ch5 */
        /* .interactive-graph, .graph-controls, .graph-info, .graph-caption, */
        /* .chapter-nav, .toc — all identical across chapters */
    </style>
</head>
<body>
    <header class="site-header">
        <div class="header-content">
            <a href="../index.html" style="text-decoration:none;color:inherit;">
                <h1 class="site-title">Microeconomics Lessons</h1>
            </a>
            <p class="site-subtitle">Chapter N: Title</p>
        </div>
    </header>

    <div class="main-container">
        <nav class="sidebar-nav">
            <ul class="nav-list">
                <li><a href="../index.html">&larr; Home</a></li>
                <li class="nav-section-title">Chapter N</li>
                <!-- One <li><a href="#id">Section Title</a></li> per section -->
            </ul>
        </nav>

        <main class="main-content">
            <!-- Sections go here -->

            <div class="chapter-nav">
                <a href="ch{N-1}-slug.html">&larr; Chapter N-1: Previous Title</a>
                <span style="color: var(--color-text-lighter);">Chapter N+1: Next Title &rarr; Coming Soon</span>
            </div>
        </main>
    </div>

    <footer class="site-footer">
        <p>&copy; 2026 Microeconomics Lessons. Created as an educational resource.</p>
    </footer>

    <script>
    <!-- All JavaScript here -->
    </script>
</body>
</html>
```

---

## 3. Pedagogical Structure

Each chapter follows this sequence:

### 3.1 The Problem (section class: `section-problem`)
- A **real-world motivating problem** that makes students ask "how does this work?"
- Concrete, vivid, relatable — ideally a named real event or policy.
- Ends with a `callout-key-insight` box previewing the analytical tools the chapter develops.
- Final paragraph: "By the end of this chapter, you will be able to…" (3–5 concrete learning outcomes).

### 3.2 Core Sections (classes: `section-intuition`, `section-geometric`)
- Typically 4–5 core sections, each following the pattern:
  - **Intuition** (`<h3>Intuition: …</h3>`) — plain-English explanation, analogies, examples.
  - **Geometric Solution** (`<h3>Geometric Solution: …</h3>`) — interactive Canvas graph with controls.
  - Prose explaining what the graph shows and connecting it to the intuition.
- Use `section-intuition` for sections that are primarily conceptual.
- Use `section-geometric` for sections centred on graphical analysis.
- Use callout boxes liberally:
  - `callout-definition` — formal definitions of new terms.
  - `callout-key-insight` — the "aha" moment.
  - `callout-example` — real-world applications or worked numerical examples.
  - `callout-warning` — common mistakes or subtle points.

### 3.3 Formal Derivations (section class: `section-mathematical`)
- **One section at the end** collecting all the rigorous mathematics.
- Full Lagrangian / optimisation setup, worked examples with specific numbers.
- Students who want intuition can stop before this section; those who want rigour can read it.
- Use display math (`\[...\]`) for key equations, inline math (`\(...\)`) in prose.

### 3.4 Summary (no special section class)
- 3–4 paragraphs recapping the key results.
- Final paragraph: a sentence bridging to the next chapter.

### Target per chapter
- **4–6 interactive Canvas graphs** (aim for 5).
- **6–8 sections** (including The Problem and Summary).
- **File size**: 55–75 KB.

---

## 4. JavaScript / Canvas Architecture

### 4.1 Overall Pattern

All JS goes in a single `<script>` block after `</footer>`. The outer wrapper is a single IIFE:

```javascript
(function() {
    'use strict';

    // 1. Colour palettes (LIGHT and DARK objects)
    // 2. Dark-mode media query listener
    // 3. Helper functions (shared across all graphs)
    // 4. Graph-specific IIFEs (one per graph)
    // 5. Global redrawAll() and resize listener
})();
```

### 4.2 Colour Palettes

Two objects, `LIGHT` and `DARK`, containing named colours for every graphical element. The active palette is stored in `let C` and swapped on `(prefers-color-scheme: dark)` change events. Always trigger `redrawAll()` on theme change.

**Standard palette keys** (extend as needed per chapter):

```javascript
const LIGHT = {
    demand: '#3b82f6', supply: '#ef4444', equilibrium: '#0a9396',
    grid: '#e5e5e5', axis: '#1a1a1a', text: '#555555', textMuted: '#aaaaaa',
    canvasBg: '#ffffff',
    ic1: '#8b5cf6', ic2: '#a78bfa', ic3: '#c4b5fd', icFaded: '#ddd6fe',
    budget: '#0a9396', budgetOld: '#aaaaaa', budgetComp: '#f59e0b',
    budgetFill: 'rgba(10, 147, 150, 0.08)',
    optimal: '#10b981', optGlow: 'rgba(16, 185, 129, 0.3)',
    se: '#10b981', ie: '#f59e0b', totalEffect: '#ef4444',
    tracePoint: '#8b5cf6', traceLine: 'rgba(139, 92, 246, 0.4)',
    // Add chapter-specific keys as needed:
    // isoquant: '#...', isocost: '#...', etc.
};
```

Dark palette mirrors light but with softer/brighter variants appropriate for dark backgrounds.

### 4.3 Helper Functions

These are defined once at the top of the script and reused by all graphs:

```javascript
// DPR-aware canvas setup — returns { ctx, w, h }
function getCtx(id) { ... }

// Generic coordinate transform: data → pixel
function tp(x, y, ox, oy, gw, gh, xMax, yMax) { ... }

// Draw axes with grid, labels, tick marks
function drawAxBase(ctx, ox, oy, gw, gh, xMax, yMax, xLabel, yLabel, xStep, yStep) { ... }

// Draw a curve given by q2 = f(q1) — used for ICs, isoquants, etc.
function drawICCurve(ctx, ox, oy, gw, gh, xMax, yMax, U, color, dashed) { ... }

// Draw a budget line / isocost line
function drawBudget(ctx, ox, oy, gw, gh, xMax, yMax, Y, p1, p2, color, dashed) { ... }

// Draw a filled dot at data coordinates
function dot(ctx, x, y, ox, oy, gw, gh, xMax, yMax, color, r) { ... }

// Draw dashed projection lines to axes
function dashedTo(ctx, x, y, ox, oy, gw, gh, xMax, yMax, color) { ... }

// Draw an arrow with arrowhead between two data points
function arrow(ctx, x1, y1, x2, y2, ox, oy, gw, gh, xMax, yMax, color, lineWidth) { ... }
```

**Adapt helpers per chapter.** For production theory, rename/refactor:
- `drawICCurve` → can be reused for isoquants (same hyperbolic shape: `q2 = f(K, L)`).
- `drawBudget` → can be reused for isocost lines.
- Add new helpers as needed (e.g., `drawProductionFn` for total/marginal/average product curves).

### 4.4 Graph-Specific IIFEs

Each interactive graph is wrapped in its own IIFE:

```javascript
(function() {
    const slider = document.getElementById('someSlider');
    const display = document.getElementById('someDisplay');
    // ... DOM references

    function draw() {
        const { ctx, w, h } = getCtx('canvasId');
        ctx.fillStyle = C.canvasBg;
        ctx.fillRect(0, 0, w, h);

        // Graph margins
        const ox = Marg.left, oy = Marg.top;
        const gw = w - ox - Marg.right, gh = h - oy - Marg.bottom;

        // Draw axes
        drawAxBase(ctx, ox, oy, gw, gh, xMax, yMax, 'xlabel', 'ylabel');

        // Draw curves, dots, labels...
    }

    // Event listeners
    slider.addEventListener('input', draw);
    // Register in global redraw list
    window._redraws = window._redraws || [];
    window._redraws.push(draw);
    draw(); // initial render
})();
```

### 4.5 Global Redraw and Resize

At the bottom of the outer IIFE:

```javascript
function redrawAll() {
    (window._redraws || []).forEach(fn => fn());
}

window.addEventListener('resize', redrawAll);
darkMQ.addEventListener('change', e => { C = e.matches ? DARK : LIGHT; redrawAll(); });
```

### 4.6 Standard Margins

```javascript
const Marg = { left: 60, top: 20, right: 20, bottom: 45 };
```

### 4.7 Canvas Dimensions

Use CSS-sized canvases with DPR scaling via `getCtx()`. Standard sizes:
- Single panel: `width="600" height="450"`
- Dual/split panel: `width="600" height="520"` (divide into top/bottom with a gap)

### 4.8 Interactivity Patterns

- **Sliders** (`<input type="range">`) for continuous parameters (prices, income, quantities).
- **Toggle buttons** for discrete states (good types, decomposition modes). Use `.active-toggle` class on the active button.
- **Trace buttons** that loop through parameter values and plot all the intermediate points.
- **Reset buttons** that clear traced state and reset sliders.
- **Info boxes** (`.graph-info`) that update dynamically with computed values.

---

## 5. CSS Classes Reference

### Section types (from style.css)
- `section-problem` — red left-border (motivating problem)
- `section-intuition` — amber left-border (conceptual development)
- `section-geometric` — blue left-border (graphical analysis)
- `section-mathematical` — purple left-border (formal maths)

### Callout boxes
- `callout callout-definition` — for formal definitions
- `callout callout-key-insight` — for key takeaways
- `callout callout-example` — for worked examples / real-world cases
- `callout callout-warning` — for common mistakes

### Graph components
- `.interactive-graph` — container for canvas + controls
- `.graph-controls` — flex row of sliders/buttons
- `.graph-info` — dynamic info panel below graph
- `.graph-caption` — italic caption below graph (Figure N.M — ...)

---

## 6. Chapter Content Plans

Reference textbook: Perloff, *Microeconomics*, 7th edition (used loosely as a structural guide, not followed closely).

### Chapter 6: Firms and Production

**The Problem:** Why do some coffee shops use expensive automated espresso machines whilst others employ skilled baristas? The choice between capital and labour depends on technology and relative input prices.

**Sections:**
1. **6.1 Production Functions** — Inputs (L, K), output (q). Short run vs long run. Total product, marginal product, average product of labour.
   - *Graph 1:* Total product of labour curve (TP_L) with diminishing returns. Slider for K level. Shows MP_L and AP_L curves below.
2. **6.2 Marginal and Average Product** — Relationship between MP and AP. MP intersects AP at AP's maximum.
   - *Graph 2:* MP_L and AP_L curves derived from TP. Highlight the crossing point.
3. **6.3 Isoquants** — Combinations of L and K that produce the same output. Analogous to indifference curves.
   - *Graph 3:* Isoquant map for Cobb-Douglas production function q = L^0.5 K^0.5. Slider for output level. Show MRTS tangent line.
4. **6.4 Returns to Scale** — Increasing, constant, decreasing. What happens when you scale all inputs proportionally?
   - *Graph 4:* Ray from origin through isoquant map. Toggle IRS/CRS/DRS to see spacing of isoquants change.
5. **6.5 Technical Progress** — How innovation shifts the production function. Neutral, labour-saving, capital-saving.
   - *Graph 5:* Isoquant shifts inward with technical progress. Toggle type.
6. **6.6 Formal Derivations** — Cobb-Douglas production, Euler's theorem, elasticity of substitution.
7. **Summary**

**Running example:** Cobb-Douglas production q = L^α K^β (with α = β = 0.5 as the default numerical case, so q = √(LK), paralleling the consumer theory utility function).

---

### Chapter 7: Costs

**The Problem:** Why did Ryanair succeed by stripping services whilst British Airways competed on quality? Different cost structures lead to different strategies.

**Sections:**
1. **7.1 Measuring Costs** — Economic vs accounting cost. Opportunity cost. Sunk costs.
2. **7.2 Short-Run Costs** — TC, FC, VC, MC, ATC, AVC, AFC. The shapes and their relationships.
   - *Graph 1:* TC/VC/FC curves (top), MC/ATC/AVC curves (bottom). Slider for output.
3. **7.3 Long-Run Costs** — All inputs variable. Isocost lines and cost minimisation (tangency with isoquant).
   - *Graph 2:* Isoquant + isocost tangency. Slider for target output level. Show expansion path.
4. **7.4 Long-Run Cost Curves** — LRAC as envelope of SRAC curves. Economies and diseconomies of scale.
   - *Graph 3:* Multiple SRAC curves with LRAC envelope. Toggle to show individual plant sizes.
5. **7.5 Learning by Doing** — How average cost falls with cumulative output.
   - *Graph 4:* Learning curve (AC vs cumulative output).
6. **7.6 Formal Derivations** — Cost minimisation via Lagrangian, Shephard's lemma, duality.
7. **Summary**

---

### Chapter 8: Competitive Firms and Markets

**The Problem:** UK dairy farmers frequently complain about being "price takers" — they must accept whatever the market offers for milk. What does it mean to be a price taker, and how does a competitive firm decide how much to produce?

**Sections:**
1. **8.1 Perfect Competition** — Characteristics, price-taking behaviour, horizontal demand curve facing the firm.
2. **8.2 Profit Maximisation** — MR = MC rule. TR, TC, and profit as functions of output.
   - *Graph 1:* TR and TC curves (top); profit curve (bottom). Slider for output. Highlight profit-maximising q.
3. **8.3 Short-Run Supply** — The firm's supply curve is its MC curve above AVC. Shutdown condition.
   - *Graph 2:* MC, ATC, AVC curves with price line. Slider for market price. Shows profit/loss rectangle. Highlight shutdown price.
4. **8.4 Long-Run Equilibrium** — Entry/exit drives economic profit to zero. P = min LRAC.
   - *Graph 3:* Firm panel + market panel side by side. Show entry/exit dynamics.
5. **8.5 Market Supply** — Horizontal summation of individual supply curves. Short-run vs long-run market supply.
   - *Graph 4:* Aggregate supply from N identical firms. Slider for N.
6. **8.6 Formal Derivations** — Profit max FOC/SOC, supply function derivation.
7. **Summary**

---

### Chapter 9: Applying the Competitive Model

**The Problem:** The EU carbon emissions trading scheme puts a price on pollution. Who bears the cost — firms or consumers? How much economic waste does it create?

**Sections:**
1. **9.1 Consumer and Producer Surplus** — Welfare measures in competitive markets.
   - *Graph 1:* CS and PS areas with adjustable supply/demand.
2. **9.2 Policies: Price Ceilings and Floors** — Rent controls, minimum wages. DWL from each.
   - *Graph 2:* Toggle ceiling/floor. Show DWL, excess demand/supply.
3. **9.3 Quotas and Tariffs** — Trade restrictions and their welfare effects.
   - *Graph 3:* Small open economy with world price, tariff slider. CS/PS/government revenue/DWL.
4. **9.4 Taxes and Subsidies Revisited** — Deeper treatment with welfare analysis.
   - *Graph 4:* Tax wedge with full welfare decomposition.
5. **9.5 Formal Derivations** — Welfare computation, Harberger triangle formula.
6. **Summary**

---

### Chapter 10: General Equilibrium and Welfare

**The Problem:** When the UK government raised fuel duty, petrol consumption fell — but car sales also dropped, public transport usage rose, and rural property prices dipped. Can we analyse an economy where everything is connected?

**Sections:**
1. **10.1 General Equilibrium in Exchange** — Edgeworth box, contract curve.
   - *Graph 1:* Edgeworth box with draggable allocation point. Show both consumers' ICs.
2. **10.2 Pareto Efficiency** — Definition, First Welfare Theorem.
   - *Graph 2:* Contract curve in the Edgeworth box. Highlight Pareto-improving region.
3. **10.3 Production and Exchange** — PPF, MRT = MRS condition.
   - *Graph 3:* PPF with indifference curve tangency.
4. **10.4 The Two Welfare Theorems** — Statement and implications.
5. **10.5 Formal Derivations** — 2×2 exchange model, Walras' law.
6. **Summary**

---

### Chapter 11: Monopoly

**The Problem:** Before 1984, British Telecom was the UK's sole telephone provider. It charged high prices and invested little in innovation. What happens when a single firm controls the entire market?

**Sections:**
1. **11.1 Sources of Monopoly** — Barriers to entry, natural monopoly, patents, network effects.
2. **11.2 Monopoly Pricing** — MR < P, MR = MC, markup over MC.
   - *Graph 1:* Demand, MR, MC curves. Profit-maximising price and quantity. Profit rectangle.
3. **11.3 Deadweight Loss of Monopoly** — Welfare comparison with perfect competition.
   - *Graph 2:* CS, PS, DWL comparison. Toggle monopoly vs competition.
4. **11.4 Natural Monopoly and Regulation** — AC pricing, MC pricing, two-part tariffs.
   - *Graph 3:* Declining AC curve with demand. Show different regulatory outcomes.
5. **11.5 Monopoly with Multiple Plants or Markets** — Multi-plant monopolist.
   - *Graph 4:* Two-plant cost allocation.
6. **11.6 Formal Derivations** — Lerner index, inverse elasticity rule, welfare loss formula.
7. **Summary**

---

### Chapter 12: Pricing and Advertising

**The Problem:** Why does a cinema charge £6 for students and £12 for adults for the same film? Why are airline tickets cheaper if booked weeks in advance?

**Sections:**
1. **12.1 First-Degree Price Discrimination** — Perfect PD, capturing all CS.
   - *Graph 1:* Demand curve with all CS extracted. Compare to uniform pricing.
2. **12.2 Third-Degree Price Discrimination** — Segmented markets with different elasticities.
   - *Graph 2:* Two-market split. Adjustable elasticities. MR₁ = MR₂ = MC condition.
3. **12.3 Second-Degree Price Discrimination** — Quantity discounts, block pricing, bundling.
   - *Graph 3:* Two-part tariff design. Show consumer types.
4. **12.4 Advertising** — Dorfman-Steiner condition. Informative vs persuasive advertising.
   - *Graph 4:* Demand shift from advertising. Optimal advertising spending.
5. **12.5 Formal Derivations** — Ramsey pricing, Dorfman-Steiner, bundling conditions.
6. **Summary**

---

### Chapter 13: Oligopoly and Monopolistic Competition

**The Problem:** The UK supermarket industry is dominated by four firms — Tesco, Sainsbury's, Asda, and Morrisons. They watch each other's prices obsessively. How do firms behave when there are only a few competitors?

**Sections:**
1. **13.1 Cournot Model** — Simultaneous quantity competition. Best-response functions.
   - *Graph 1:* Best-response curves with Nash equilibrium. Adjustable costs.
2. **13.2 Stackelberg Model** — Sequential quantity competition. First-mover advantage.
   - *Graph 2:* Leader's residual demand. Compare Cournot vs Stackelberg outcomes.
3. **13.3 Bertrand Model** — Price competition. Bertrand paradox.
   - *Graph 3:* Price undercutting dynamics. Show convergence to MC.
4. **13.4 Monopolistic Competition** — Product differentiation, short-run profits, long-run zero profit.
   - *Graph 4:* Firm's demand tangent to LRAC in long-run equilibrium.
5. **13.5 Collusion and Cartels** — Joint profit maximisation, incentive to cheat.
   - *Graph 5:* Cartel outcome vs defection payoff.
6. **13.6 Formal Derivations** — Cournot/Stackelberg/Bertrand equilibrium derivation, Chamberlin model.
7. **Summary**

---

### Chapter 14: Game Theory

**The Problem:** During the Cold War, the US and USSR each faced a choice: build more nuclear weapons or negotiate disarmament. Each side's best move depended on what the other did. This is the logic of strategic interaction.

**Sections:**
1. **14.1 Normal-Form Games** — Players, strategies, payoffs. Dominant strategies.
   - *Graph 1:* Interactive payoff matrix. Click to highlight best responses. Show dominant strategy if it exists.
2. **14.2 Nash Equilibrium** — Definition, finding NE in pure and mixed strategies.
   - *Graph 2:* Best-response diagram for mixed strategies. Show NE as intersection.
3. **14.3 Sequential Games** — Game trees, backward induction, subgame perfect equilibrium.
   - *Graph 3:* Interactive game tree with backward induction highlighting.
4. **14.4 Repeated Games** — Folk theorem, trigger strategies, cooperation.
   - *Graph 4:* Repeated PD payoff streams. Discount factor slider. Show when cooperation is sustainable.
5. **14.5 Applications** — Entry deterrence, bargaining, auction design.
6. **14.6 Formal Derivations** — Mixed strategy NE computation, SPE, discounted payoff sums.
7. **Summary**

---

### Chapter 15: Factor Markets

**The Problem:** Why are Premier League footballers paid millions whilst nurses earn £35,000? The answer lies in the demand for and supply of different types of labour.

**Sections:**
1. **15.1 Demand for Labour** — MRP_L = MR × MP_L. Derived demand.
   - *Graph 1:* MRP curve as labour demand. Slider for output price.
2. **15.2 Supply of Labour** — Income-leisure trade-off. Backward-bending supply curve.
   - *Graph 2:* Labour-leisure choice with budget constraint. Show backward bend at high wages.
3. **15.3 Labour Market Equilibrium** — Competitive labour market. Monopsony.
   - *Graph 3:* Competitive vs monopsony equilibrium. DWL of monopsony.
4. **15.4 Capital Markets** — Rental rate of capital, investment decisions.
   - *Graph 4:* Firm's demand for capital. User cost of capital.
5. **15.5 Formal Derivations** — MRP derivation, monopsony wage-setting.
6. **Summary**

---

### Chapter 16: Interest Rates, Investments, and Capital Markets

**The Problem:** Should a university invest £2 million in a new laboratory that will generate research income for 20 years? The answer depends on the interest rate and the time value of money.

**Sections:**
1. **16.1 Present Value** — Discounting, NPV, compounding.
   - *Graph 1:* PV of a stream of payments. Interest rate slider. Show how PV shrinks.
2. **16.2 Investment Decisions** — NPV rule. IRR. Comparing projects.
   - *Graph 2:* NPV as function of discount rate. Show IRR crossing.
3. **16.3 Intertemporal Choice** — Two-period consumption model. Borrowing and lending.
   - *Graph 3:* Intertemporal budget constraint with indifference curves. Interest rate slider.
4. **16.4 Bond Pricing and the Yield Curve** — How bond prices relate to interest rates.
   - *Graph 4:* Bond price vs yield. Duration and convexity.
5. **16.5 Formal Derivations** — Fisher equation, optimal intertemporal allocation, bond math.
6. **Summary**

---

### Chapter 17: Uncertainty

**The Problem:** Should you buy travel insurance for a £3,000 holiday? The expected loss from cancellation might be only £150, but the insurance costs £80. Why might a rational person still buy it?

**Sections:**
1. **17.1 Expected Value and Expected Utility** — Why EV maximisation fails. EU theory.
   - *Graph 1:* Concave utility function. Show EU < U(EV) for risk-averse agent.
2. **17.2 Risk Aversion** — Concavity, certainty equivalent, risk premium.
   - *Graph 2:* Utility function with CE and RP marked. Slider for probability. Toggle concave/convex/linear.
3. **17.3 Insurance** — Fair insurance, willingness to pay. Moral hazard preview.
   - *Graph 3:* State-contingent consumption diagram. 45° line. Insurance moves toward certainty.
4. **17.4 Diversification** — Portfolio theory basics. Don't put all eggs in one basket.
   - *Graph 4:* Risk-return frontier with two assets. Correlation slider.
5. **17.5 Formal Derivations** — Arrow-Pratt measures, optimal insurance, CAPM basics.
6. **Summary**

---

### Chapter 18: Externalities and Public Goods

**The Problem:** London's Ultra Low Emission Zone charges polluting vehicles £12.50/day to enter. Why can't the market handle pollution on its own? What's the economic justification for government intervention?

**Sections:**
1. **18.1 Externalities** — Private vs social cost/benefit. Negative and positive externalities.
   - *Graph 1:* Supply = PMC, social cost = SMC. Overproduction. DWL triangle.
2. **18.2 Solutions: Taxes and Subsidies** — Pigovian tax sets P = SMC.
   - *Graph 2:* Pigovian tax correcting the externality. Slider for tax rate. Show optimal tax.
3. **18.3 The Coase Theorem** — Bargaining and property rights. Transaction costs.
   - *Graph 3:* Bilateral bargaining diagram. Show efficient outcome regardless of rights assignment.
4. **18.4 Public Goods** — Non-rivalry, non-excludability. Free-rider problem.
   - *Graph 4:* Vertical summation of demand curves. Show underprovision by the market.
5. **18.5 Formal Derivations** — Samuelson condition, Lindahl pricing, optimal Pigovian tax.
6. **Summary**

---

### Chapter 19: Asymmetric Information

**The Problem:** When buying a used car, the seller knows the car's history but the buyer does not. George Akerlof showed this can cause the market to collapse entirely — the famous "market for lemons."

**Sections:**
1. **19.1 Adverse Selection** — Hidden information before the transaction. Lemons problem.
   - *Graph 1:* Quality distribution. Show how high-quality sellers exit. Market unravelling.
2. **19.2 Signalling** — Spence's job-market signalling. Education as a signal.
   - *Graph 2:* Separating equilibrium. Cost of signal for high vs low type.
3. **19.3 Screening** — Insurance markets. Menu of contracts.
   - *Graph 3:* Two contract offers in state-contingent space. IC of high/low risk types.
4. **19.4 Moral Hazard** — Hidden action after the transaction. Principal-agent problem.
   - *Graph 4:* Effort choice under different contract structures. Fixed wage vs performance pay.
5. **19.5 Formal Derivations** — Akerlof model, Spence signalling game, Rothschild-Stiglitz.
6. **Summary**

---

### Chapter 20: Contracts and Moral Hazards

**The Problem:** When a football club hires a new manager, how should it structure the contract? A fixed salary gives security but no incentive for results; pure performance pay creates motivation but exposes the manager to risk.

**Sections:**
1. **20.1 The Principal-Agent Problem** — Hidden action, risk sharing, incentive compatibility.
   - *Graph 1:* Agent's utility under different contract types. Participation and incentive constraints.
2. **20.2 Optimal Contract Design** — Linear contracts, piece rates, bonuses.
   - *Graph 2:* Optimal incentive intensity. Trade-off between risk and incentives.
3. **20.3 Efficiency Wages** — Paying above market-clearing wage to reduce shirking.
   - *Graph 3:* No-shirking condition. Equilibrium unemployment.
4. **20.4 Team Production and Tournaments** — Free-riding in teams. Rank-order tournaments.
   - *Graph 4:* Tournament payoffs. Optimal prize spread.
5. **20.5 Formal Derivations** — First-best vs second-best contracts, Holmström's informativeness principle.
6. **Summary**

---

## 7. Progress Tracker

### Completed Chapters (DO NOT rebuild these)

| Ch | Title | File | Graphs | Size |
|----|-------|------|--------|------|
| 2 | Supply and Demand | ch2-supply-and-demand.html | 5 | ~65 KB |
| 3 | Applying Supply and Demand | ch3-applying-supply-and-demand.html | 5 | ~65 KB |
| 4 | Consumer Choice | ch4-consumer-choice.html | 5 | ~70 KB |
| 5 | Applying Consumer Theory | ch5-applying-consumer-theory.html | 5 | ~68 KB |
| 6 | Firms and Production | ch6-firms-and-production.html | 5 | 72 KB |
| 7 | Costs | ch7-costs.html | 4 | 66 KB |
| 8 | Competitive Firms and Markets | ch8-competitive-firms-and-markets.html | 4 | 67 KB |
| 9 | Applying the Competitive Model | ch9-applying-the-competitive-model.html | 4 | 66 KB |
| 10 | General Equilibrium and Welfare | ch10-general-equilibrium-and-welfare.html | 4 | 73 KB |
| 11 | Monopoly | ch11-monopoly.html | 4 | 68 KB |
| 12 | Pricing and Advertising | ch12-pricing-and-advertising.html | 4 | 65 KB |
| 13 | Oligopoly and Monopolistic Competition | ch13-oligopoly-and-monopolistic-competition.html | 5 | 71 KB |
| 14 | Game Theory | ch14-game-theory.html | 4 | 70 KB |
| 15 | Factor Markets | ch15-factor-markets.html | 4 | 54 KB |
| 16 | Interest Rates, Investments, and Capital Markets | ch16-interest-rates-investments-and-capital-markets.html | 4 | 54 KB |

**All 15 chapters above are fully linked:** index.html cards are active, and each chapter's forward/backward navigation links work. Chapter 1 is "Coming soon" (no content planned yet).

### Remaining Chapters (build these next)

- **Chapter 17:** Uncertainty
- **Chapter 18:** Externalities and Public Goods
- **Chapter 19:** Asymmetric Information
- **Chapter 20:** Contracts and Moral Hazards

### Key Bug Fix to Remember

In Chapter 8, a `started = false;` assignment without `var` in `'use strict'` mode caused a ReferenceError that silently killed Graphs 3 and 4. **Always use `var` declarations in strict mode.** Check for this in every chapter with `grep -nP '^\s+started\s*=' FILE`.

---

## 8. Running Examples and Continuity

- **Chapters 4–5** use Cobb-Douglas utility: U = q₁ · q₂, with Y = 30, p₁ = 5, p₂ = 3.
- **Chapters 6–7** should use Cobb-Douglas production: q = L^α K^β (default α = β = 0.5, so q = √(LK)). This parallels the consumer theory structure and lets students see the duality.
- **Chapter 8 onwards** can introduce linear demand/supply for market-level analysis (P = a − bQ, etc.) whilst keeping Cobb-Douglas for firm-level cost derivations.
- Real-world examples should be predominantly **UK-based** (consistent with £ currency used throughout).

---

## 9. Quality Checklist (run after each chapter)

```bash
cd "micro/chapters"
FILE="ch{N}-{slug}.html"

echo "=== File size ==="
ls -la "$FILE"

echo "=== Section tags ==="
grep -c '<section' "$FILE"
grep -c '</section>' "$FILE"

echo "=== Canvas elements ==="
grep -c '<canvas' "$FILE"

echo "=== Links check ==="
grep -oP 'href="[^"]*"' "$FILE"

echo "=== MathJax loaded? ==="
grep -c 'mathjax' "$FILE"

echo "=== IIFE count ==="
grep -c '(function()' "$FILE"
```

Expectations:
- Section open/close counts match (6–8 each)
- Canvas count: 4–6
- MathJax: at least 1
- IIFE count: canvas count + 1 (outer wrapper)
- File size: 55–75 KB

---

## 10. Style and Tone

- **British English** throughout (colour, labour, minimise, optimise, behaviour, defence).
- Second person where appropriate ("As you raise the price slider…").
- Formal but accessible — academic tone without being stuffy.
- Currency: **£** (pounds sterling).
- Figure captions: "Figure N.M — Description."
- No emojis in chapter content.
- LaTeX conventions: `\text{}` for labels in equations, `\quad` for spacing, `\Rightarrow` for implications.
