<!DOCTYPE html>
<html>
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <script src="https://aframe.io/releases/1.4.0/aframe.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/mind-ar@1.2.1/dist/mindar-image-aframe.prod.js"></script>
  <style>
    #info-panel {
      position: fixed;
      top: 10px;
      left: 10px;
      background: rgba(0, 0, 0, 0.8);
      color: white;
      padding: 15px;
      border-radius: 10px;
      font-family: Arial, sans-serif;
      max-width: 300px;
      z-index: 1000;
    }
    
    #info-panel h3 {
      margin: 0 0 10px 0;
      color: #4CAF50;
    }
    
    .metric {
      margin: 5px 0;
      padding: 5px;
      background: rgba(255, 255, 255, 0.1);
      border-radius: 5px;
    }
    
    .triangle {
      color: #FFD700;
      font-weight: bold;
    }
    
    .quadrilateral {
      color: #00BFFF;
      font-weight: bold;
    }
  </style>
</head>
<body>
  <div id="info-panel">
    <h3>📐 Геометричні обчислення</h3>
    <div id="calculations">
      <div class="metric">Видимі маркери: <span id="visible-markers">-</span></div>
      <div class="metric quadrilateral">Периметр чотирикутника: <span id="quad-perimeter">-</span></div>
      <div class="metric triangle">▲ ABC: площа <span id="triangle-abc">-</span></div>
      <div class="metric triangle">▲ ACD: площа <span id="triangle-acd">-</span></div>
      <div class="metric triangle">▲ ABD: площа <span id="triangle-abd">-</span></div>
      <div class="metric triangle">▲ BCD: площа <span id="triangle-bcd">-</span></div>
    </div>
  </div>

  <a-scene mindar-image="imageTargetSrc: ./targets.mind; maxTrack: 3;"
           vr-mode-ui="enabled: false"
           device-orientation-permission-ui="enabled: false"
           quadrilateral-system>
    
    <a-camera position="0 0 0" look-controls="enabled: false"></a-camera>
    
    <!-- Маркери з 3D об'єктами -->
    <a-entity id="marker-a" mindar-image-target="targetIndex: 0">
      <a-sphere color="#e74c3c" radius="0.1" position="0 0.1 0"></a-sphere>
      <a-text value="A" position="0 0.3 0" align="center" color="white" 
              geometry="primitive: plane; width: 0.3; height: 0.15" 
              material="color: #e74c3c; transparent: true; opacity: 0.8"></a-text>
    </a-entity>
    
    <a-entity id="marker-b" mindar-image-target="targetIndex: 1">
      <a-sphere color="#3498db" radius="0.1" position="0 0.1 0"></a-sphere>
      <a-text value="B" position="0 0.3 0" align="center" color="white"
              geometry="primitive: plane; width: 0.3; height: 0.15" 
              material="color: #3498db; transparent: true; opacity: 0.8"></a-text>
    </a-entity>
    
    <a-entity id="marker-c" mindar-image-target="targetIndex: 2">
      <a-sphere color="#9b59b6" radius="0.1" position="0 0.1 0"></a-sphere>
      <a-text value="C" position="0 0.3 0" align="center" color="white"
              geometry="primitive: plane; width: 0.3; height: 0.15" 
              material="color: #9b59b6; transparent: true; opacity: 0.8"></a-text>
    </a-entity>
    
    <a-entity id="marker-d" mindar-image-target="targetIndex: 3">
      <a-sphere color="#2ecc71" radius="0.1" position="0 0.1 0"></a-sphere>
      <a-text value="D" position="0 0.3 0" align="center" color="white"
              geometry="primitive: plane; width: 0.3; height: 0.15" 
              material="color: #2ecc71; transparent: true; opacity: 0.8"></a-text>
    </a-entity>
  </a-scene>

  <script>
    // Компонент для управління системою чотирикутника
    AFRAME.registerComponent('quadrilateral-system', {
      init: function() {
        // Ініціалізація маркерів
        this.markers = {
          'a': document.getElementById('marker-a'),
          'b': document.getElementById('marker-b'),
          'c': document.getElementById('marker-c'),
          'd': document.getElementById('marker-d')
        };
        
        // Відстеження видимості маркерів
        this.markerVisible = {
          'a': false, 'b': false, 'c': false, 'd': false
        };
        
        // Створення ліній для чотирикутника (A-B-C-D-A)
        this.quadLines = {
          'ab': this.createLine('#e74c3c'),
          'bc': this.createLine('#3498db'),
          'cd': this.createLine('#9b59b6'),
          'da': this.createLine('#2ecc71')
        };
        
        // Створення діагоналей (для розрахунку трикутників)
        this.diagonals = {
          'ac': this.createLine('#f39c12', true),
          'bd': this.createLine('#e67e22', true)
        };
        
        // Встановлення обробників подій
        this.setupEvents();
        
        // Елементи UI для відображення результатів
        this.uiElements = {
          visibleMarkers: document.getElementById('visible-markers'),
          quadPerimeter: document.getElementById('quad-perimeter'),
          triangleABC: document.getElementById('triangle-abc'),
          triangleACD: document.getElementById('triangle-acd'),
          triangleABD: document.getElementById('triangle-abd'),
          triangleBCD: document.getElementById('triangle-bcd')
        };
      },
      
      createLine: function(color, isDiagonal = false) {
        const line = document.createElement('a-cylinder');
        line.setAttribute('radius', '0.02');
        line.setAttribute('height', '1');
        line.setAttribute('material', `color: ${color}; transparent: true; opacity: ${isDiagonal ? '0.5' : '0.8'}`);
        line.setAttribute('visible', false);
        
        document.querySelector('a-scene').appendChild(line);
        return line;
      },
      
      setupEvents: function() {
        Object.keys(this.markers).forEach(id => {
          const marker = this.markers[id];
          
          marker.addEventListener("targetFound", () => {
            this.markerVisible[id] = true;
            this.updateVisualization();
          });
          
          marker.addEventListener("targetLost", () => {
            this.markerVisible[id] = false;
            this.updateVisualization();
          });
        });
      },
      
      updateVisualization: function() {
        const visibleCount = Object.values(this.markerVisible).filter(v => v).length;
        const visibleIds = Object.keys(this.markerVisible).filter(id => this.markerVisible[id]);
        
        // Оновлення UI
        this.uiElements.visibleMarkers.textContent = `${visibleCount}/4 (${visibleIds.join(', ').toUpperCase()})`;
        
        // Показати лінії чотирикутника тільки якщо всі 4 маркери видимі
        const showQuad = visibleCount === 4;
        
        Object.values(this.quadLines).forEach(line => {
          line.setAttribute('visible', showQuad);
        });
        
        Object.values(this.diagonals).forEach(line => {
          line.setAttribute('visible', showQuad);
        });
        
        if (showQuad) {
          this.calculateAndDisplay();
        } else {
          this.resetCalculations();
        }
      },
      
      calculateAndDisplay: function() {
        // Отримання позицій маркерів у світових координатах
        const positions = {};
        Object.keys(this.markers).forEach(id => {
          positions[id] = new THREE.Vector3();
          this.markers[id].object3D.getWorldPosition(positions[id]);
        });
        
        // Розрахунок периметру чотирикутника (A→B→C→D→A)
        const perimeter = this.calculateQuadrilateralPerimeter(positions);
        this.uiElements.quadPerimeter.textContent = `${perimeter.toFixed(2)} м`;
        
        // Розрахунок площ усіх можливих трикутників
        const triangles = [
          { name: 'ABC', points: [positions.a, positions.b, positions.c], element: this.uiElements.triangleABC },
          { name: 'ACD', points: [positions.a, positions.c, positions.d], element: this.uiElements.triangleACD },
          { name: 'ABD', points: [positions.a, positions.b, positions.d], element: this.uiElements.triangleABD },
          { name: 'BCD', points: [positions.b, positions.c, positions.d], element: this.uiElements.triangleBCD }
        ];
        
        triangles.forEach(triangle => {
          const area = this.calculateTriangleArea(triangle.points[0], triangle.points[1], triangle.points[2]);
          triangle.element.textContent = `${area.toFixed(3)} м²`;
        });
      },
      
      calculateQuadrilateralPerimeter: function(positions) {
        const sideAB = positions.a.distanceTo(positions.b);
        const sideBC = positions.b.distanceTo(positions.c);
        const sideCD = positions.c.distanceTo(positions.d);
        const sideDA = positions.d.distanceTo(positions.a);
        
        return sideAB + sideBC + sideCD + sideDA;
      },
      
      calculateTriangleArea: function(pointA, pointB, pointC) {
        // Використовуємо формулу Герона для розрахунку площі трикутника
        const sideA = pointB.distanceTo(pointC);
        const sideB = pointC.distanceTo(pointA);
        const sideC = pointA.distanceTo(pointB);
        
        const semiPerimeter = (sideA + sideB + sideC) / 2;
        
        const area = Math.sqrt(
          semiPerimeter * 
          (semiPerimeter - sideA) * 
          (semiPerimeter - sideB) * 
          (semiPerimeter - sideC)
        );
        
        return isNaN(area) ? 0 : area;
      },
      
      resetCalculations: function() {
        this.uiElements.quadPerimeter.textContent = '-';
        this.uiElements.triangleABC.textContent = '-';
        this.uiElements.triangleACD.textContent = '-';
        this.uiElements.triangleABD.textContent = '-';
        this.uiElements.triangleBCD.textContent = '-';
      },
      
      updateLinePosition: function(line, pointA, pointB) {
        // Розрахунок відстані між точками
        const distance = pointA.distanceTo(pointB);
        
        // Встановлення висоти циліндра
        line.setAttribute('height', distance);
        
        // Розміщення лінії посередині між точками
        const midPoint = {
          x: (pointA.x + pointB.x) / 2,
          y: (pointA.y + pointB.y) / 2,
          z: (pointA.z + pointB.z) / 2
        };
        line.setAttribute('position', midPoint);
        
        // Орієнтація лінії
        line.object3D.lookAt(pointB);
        line.object3D.rotateX(Math.PI / 2);
      },
      
      tick: function() {
        // Оновлення позицій ліній якщо всі маркери видимі
        if (Object.values(this.markerVisible).every(v => v)) {
          const positions = {};
          Object.keys(this.markers).forEach(id => {
            positions[id] = new THREE.Vector3();
            this.markers[id].object3D.getWorldPosition(positions[id]);
          });
          
          // Оновлення ліній чотирикутника
          this.updateLinePosition(this.quadLines.ab, positions.a, positions.b);
          this.updateLinePosition(this.quadLines.bc, positions.b, positions.c);
          this.updateLinePosition(this.quadLines.cd, positions.c, positions.d);
          this.updateLinePosition(this.quadLines.da, positions.d, positions.a);
          
          // Оновлення діагоналей
          this.updateLinePosition(this.diagonals.ac, positions.a, positions.c);
          this.updateLinePosition(this.diagonals.bd, positions.b, positions.d);
        }
      }
    });
  </script>
</body>
</html>
