# Graphify.java
the given data will be shown in graphicaly

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Graphify – Runtime Object Visualizer</title>
  <script src="https://d3js.org/d3.v7.min.js"></script>
  <style>
    /* ===== MATRIX BACKGROUND ===== */
    body {
      margin: 0;
      overflow-x: hidden;
      font-family: "Consolas", monospace;
      color: #00ff00;
      background: black;
    }

    canvas {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      z-index: 0;
    }

    /* ===== CONTAINER ===== */
    .container {
      position: relative;
      z-index: 1;
      max-width: 900px;
      margin: 40px auto;
      background: rgba(0, 0, 0, 0.8);
      padding: 25px;
      border-radius: 10px;
      border: 1px solid #00ff00;
      text-align: center;
      box-shadow: 0 0 20px #00ff00;
    }

    h1 {
      color: #00ff00;
      text-shadow: 0 0 10px #00ff00;
      margin-bottom: 20px;
    }

    input, select, button {
      display: block;
      width: 80%;
      margin: 10px auto;
      background: black;
      color: #00ff00;
      border: 1px solid #00ff00;
      border-radius: 5px;
      padding: 10px;
      font-size: 15px;
      transition: 0.3s;
    }

    input:focus, select:focus {
      outline: none;
      box-shadow: 0 0 10px #00ff00;
    }

    button {
      cursor: pointer;
      width: 50%;
    }

    button:hover {
      background: #003300;
      box-shadow: 0 0 10px #00ff00;
    }

    svg {
      width: 100%;
      height: 70vh;
      margin-top: 20px;
      border-top: 1px solid #00ff00;
      border-radius: 5px;
    }

    .relation-list {
      color: #00ffff;
      margin-top: 10px;
      font-size: 14px;
    }

    .relation-item {
      margin: 5px 0;
    }

    h2 {
      border-bottom: 1px solid #00ff00;
      display: inline-block;
      padding-bottom: 3px;
      margin-top: 30px;
    }
  </style>
</head>
<body>
  <canvas id="matrix"></canvas>

  <div class="container">
    <h1>🧠 Graphify – Interactive Runtime Object Visualizer</h1>

    <!-- STEP 1 -->
    <h2>1️⃣ Create Objects</h2>
    <input type="text" id="objectName" placeholder="Enter Object Name (e.g., obj1)" />
    <input type="text" id="objectType" placeholder="Enter Class Name (e.g., Student)" />
    <button onclick="addObject()">Add Object</button>

    <!-- STEP 2 -->
    <h2>2️⃣ Define & Manage Relationships</h2>
    <select id="sourceSelect"></select>
    <select id="targetSelect"></select>
    <button onclick="addRelation()">Add Relationship</button>

    <select id="relationSelect"></select>
    <button onclick="deleteRelation()">Delete Selected Relationship</button>

    <div class="relation-list" id="relationList"></div>

    <!-- STEP 3 -->
    <h2>3️⃣ Visualize Graph</h2>
    <button onclick="visualizeGraph()">Visualize Object Graph</button>

    <svg id="graph"></svg>
  </div>

  <script>
    /* ===== MATRIX RAIN ===== */
    const canvas = document.getElementById("matrix");
    const ctx = canvas.getContext("2d");
    canvas.height = window.innerHeight;
    canvas.width = window.innerWidth;

    const letters = "ABCDEFGHIJKLMNOPQRSTUVWXYZ123456789@#$%^&*()";
    const fontSize = 16;
    const columns = canvas.width / fontSize;
    const drops = Array(Math.floor(columns)).fill(1);

    function drawMatrix() {
      ctx.fillStyle = "rgba(0, 0, 0, 0.05)";
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      ctx.fillStyle = "#00ff00";
      ctx.font = fontSize + "px monospace";

      for (let i = 0; i < drops.length; i++) {
        const text = letters.charAt(Math.floor(Math.random() * letters.length));
        ctx.fillText(text, i * fontSize, drops[i] * fontSize);
        if (drops[i] * fontSize > canvas.height && Math.random() > 0.975) {
          drops[i] = 0;
        }
        drops[i]++;
      }
    }
    setInterval(drawMatrix, 40);

    window.addEventListener("resize", () => {
      canvas.height = window.innerHeight;
      canvas.width = window.innerWidth;
    });

    /* ===== GRAPHIFY LOGIC ===== */
    let objects = [];
    let relationships = [];

    function updateDropdowns() {
      const source = document.getElementById("sourceSelect");
      const target = document.getElementById("targetSelect");
      source.innerHTML = "";
      target.innerHTML = "";
      objects.forEach((obj) => {
        source.innerHTML += <option value="${obj.id}">${obj.id}</option>;
        target.innerHTML += <option value="${obj.id}">${obj.id}</option>;
      });
    }

    function updateRelationList() {
      const listDiv = document.getElementById("relationList");
      const relationSelect = document.getElementById("relationSelect");
      listDiv.innerHTML = "";
      relationSelect.innerHTML = "";

      if (relationships.length === 0) {
        listDiv.innerHTML = "<i>No relationships defined</i>";
        return;
      }

      relationships.forEach((r, i) => {
        const text = ${r.source} → ${r.target};
        listDiv.innerHTML += <div class='relation-item'>${i + 1}. ${text}</div>;
        relationSelect.innerHTML += <option value="${i}">${text}</option>;
      });
    }

    function addObject() {
      const name = document.getElementById("objectName").value.trim();
      const type = document.getElementById("objectType").value.trim();
      if (!name || !type) {
        alert("⚠ Please enter both Object Name and Class Type!");
        return;
      }
      if (objects.some((o) => o.id === name)) {
        alert("⚠ Object name already exists!");
        return;
      }
      objects.push({ id: name, type: "Class: " + type });
      updateDropdowns();
      document.getElementById("objectName").value = "";
      document.getElementById("objectType").value = "";
      alert(✅ Object "${name}" of type "${type}" created.);
    }

    function addRelation() {
      const source = document.getElementById("sourceSelect").value;
      const target = document.getElementById("targetSelect").value;
      if (!source || !target) return alert("⚠ Add at least two objects first!");
      if (source === target) {
        alert("⚠ Source and Target cannot be same!");
        return;
      }
      relationships.push({ source, target });
      updateRelationList();
      alert(🔗 Relationship created: ${source} → ${target});
    }

    function deleteRelation() {
      const index = document.getElementById("relationSelect").value;
      if (index === "" || relationships.length === 0) {
        alert("⚠ No relationship selected!");
        return;
      }
      const removed = relationships.splice(index, 1);
      alert(❌ Deleted relationship: ${removed[0].source} → ${removed[0].target});
      updateRelationList();
    }

    function visualizeGraph() {
      if (objects.length === 0) {
        alert("⚠ Please create at least one object first!");
        return;
      }

      const svg = d3.select("#graph");
      svg.selectAll("*").remove();

      const width = svg.node().getBoundingClientRect().width;
      const height = svg.node().getBoundingClientRect().height;

      const simulation = d3
        .forceSimulation(objects)
        .force("link", d3.forceLink(relationships).id((d) => d.id).distance(150))
        .force("charge", d3.forceManyBody().strength(-400))
        .force("center", d3.forceCenter(width / 2, height / 2));

      const link = svg
        .append("g")
        .selectAll("line")
        .data(relationships)
        .enter()
        .append("line")
        .attr("stroke", "#00ff00")
        .attr("stroke-width", 2)
        .attr("opacity", 0.8);

      const node = svg
        .append("g")
        .selectAll("circle")
        .data(objects)
        .enter()
        .append("circle")
        .attr("r", 30)
        .attr("fill", "#00cc66")
        .attr("stroke", "#00ff00")
        .attr("stroke-width", 2)
        .call(
          d3
            .drag()
            .on("start", dragStarted)
            .on("drag", dragged)
            .on("end", dragEnded)
        )
        .on("mouseover", function () {
          d3.select(this).attr("fill", "#00ffff");
        })
        .on("mouseout", function () {
          d3.select(this).attr("fill", "#00cc66");
        });

      const label = svg
        .append("g")
        .selectAll("text")
        .data(objects)
        .enter()
        .append("text")
        .attr("fill", "#00ff00")
        .attr("font-size", 14)
        .attr("text-anchor", "middle")
        .attr("dy", 5)
        .text((d) => d.id);

      simulation.on("tick", () => {
        link
          .attr("x1", (d) => d.source.x)
          .attr("y1", (d) => d.source.y)
          .attr("x2", (d) => d.target.x)
          .attr("y2", (d) => d.target.y);

        node.attr("cx", (d) => d.x).attr("cy", (d) => d.y);
        label.attr("x", (d) => d.x).attr("y", (d) => d.y);
      });

      function dragStarted(event, d) {
        if (!event.active) simulation.alphaTarget(0.3).restart();
        d.fx = d.x;
        d.fy = d.y;
      }

      function dragged(event, d) {
        d.fx = event.x;
        d.fy = event.y;
      }

      function dragEnded(event, d) {
        if (!event.active) simulation.alphaTarget(0);
        d.fx = null;
        d.fy = null;
      }
    }

    updateRelationList(); // initialize
  </script>
</body>
</html>
