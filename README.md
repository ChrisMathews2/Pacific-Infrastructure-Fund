Pacific-Infrastructure-Fund
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Editable System Architecture Dashboard</title>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
            margin: 0;
            padding: 0;
            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
            min-height: 100vh;
            display: flex;
            flex-direction: column;
        }

        .dashboard-header {
            background: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(10px);
            padding: 20px;
            text-align: center;
            border-bottom: 1px solid rgba(255, 255, 255, 0.2);
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .dashboard-header h1 {
            color: white;
            margin: 0;
            font-size: 2em;
            font-weight: 300;
            letter-spacing: 2px;
        }

        .toolbar {
            display: flex;
            gap: 10px;
        }

        .toolbar button {
            background: rgba(255, 255, 255, 0.2);
            border: 1px solid rgba(255, 255, 255, 0.3);
            color: white;
            padding: 10px 20px;
            border-radius: 25px;
            cursor: pointer;
            transition: all 0.3s ease;
            font-size: 14px;
            backdrop-filter: blur(10px);
        }

        .toolbar button:hover {
            background: rgba(255, 255, 255, 0.3);
            transform: translateY(-2px);
        }

        .toolbar button.active {
            background: #2196F3;
            border-color: #2196F3;
        }

        .main-container {
            flex: 1;
            display: flex;
            padding: 20px;
            gap: 20px;
            position: relative;
        }

        .diagram-container {
            flex: 1;
            background: rgba(255, 255, 255, 0.95);
            border-radius: 20px;
            padding: 20px;
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.2);
            position: relative;
            overflow: hidden;
            cursor: crosshair;
        }

        .diagram-container.connecting {
            cursor: crosshair;
        }

        .info-panel {
            width: 350px;
            background: rgba(255, 255, 255, 0.95);
            border-radius: 20px;
            padding: 20px;
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.2);
            overflow-y: auto;
            max-height: calc(100vh - 120px);
            transition: all 0.3s ease;
        }

        .info-panel.collapsed {
            width: 0;
            padding: 0;
            opacity: 0;
            margin-right: -350px;
        }

        .panel-toggle {
            position: absolute;
            right: 370px;
            top: 50%;
            transform: translateY(-50%);
            background: #2196F3;
            color: white;
            border: none;
            padding: 10px 5px;
            border-radius: 5px 0 0 5px;
            cursor: pointer;
            z-index: 10;
            transition: all 0.3s ease;
        }

        .panel-toggle.panel-collapsed {
            right: 20px;
            border-radius: 0 5px 5px 0;
        }

        .diagram-container.expanded {
            margin-right: -330px;
        }

        .component {
            position: absolute;
            background: white;
            border: 2px solid #333;
            border-radius: 10px;
            padding: 15px 25px;
            cursor: move;
            transition: all 0.3s ease;
            font-weight: 500;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
            user-select: none;
            min-width: 100px;
            text-align: center;
        }

        .component:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
            border-color: #2196F3;
        }

        .component.active {
            background: #2196F3;
            color: white;
            border-color: #1976D2;
        }

        .component.editing {
            cursor: text;
            user-select: text;
        }

        .component .delete-btn {
            position: absolute;
            top: -10px;
            right: -10px;
            background: #f44336;
            color: white;
            border: none;
            border-radius: 50%;
            width: 25px;
            height: 25px;
            cursor: pointer;
            display: none;
            font-size: 12px;
            line-height: 25px;
            text-align: center;
        }

        .component:hover .delete-btn {
            display: block;
        }

        .connection {
            stroke: #666;
            stroke-width: 3;
            fill: none;
            opacity: 0.7;
            transition: all 0.3s ease;
            cursor: pointer;
        }

        .connection:hover {
            stroke-width: 5;
            opacity: 1;
        }

        .connection.highlighted {
            stroke: #2196F3;
            stroke-width: 5;
            opacity: 1;
        }

        .arrow {
            fill: #666;
            transition: all 0.3s ease;
        }

        .arrow.highlighted {
            fill: #2196F3;
        }

        .arrow-marker {
            fill: #666;
        }

        .arrow-marker-highlighted {
            fill: #2196F3;
        }

        .info-panel h2 {
            margin-top: 0;
            color: #333;
            font-size: 1.5em;
            margin-bottom: 20px;
        }

        .component-details-wrapper {
            transition: all 0.3s ease;
            overflow: hidden;
            max-height: 2000px;
        }

        .component-details-wrapper.collapsed {
            max-height: 0;
            margin-top: -20px;
            opacity: 0;
        }

        .collapsible-section {
            margin-bottom: 20px;
            border: 1px solid #e0e0e0;
            border-radius: 10px;
            overflow: hidden;
            background: white;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.05);
        }

        .collapsible-header {
            padding: 15px;
            background: #f8f9fa;
            cursor: pointer;
            display: flex;
            justify-content: space-between;
            align-items: center;
            transition: all 0.3s ease;
            user-select: none;
        }

        .collapsible-header:hover {
            background: #e9ecef;
        }

        .collapsible-header h3 {
            margin: 0;
            color: #2196F3;
            font-size: 1.1em;
            font-weight: 600;
        }

        .collapse-icon {
            font-size: 1.2em;
            color: #666;
            transition: transform 0.3s ease;
        }

        .collapsible-header.collapsed .collapse-icon {
            transform: rotate(-90deg);
        }

        .collapsible-content {
            max-height: 1000px;
            overflow: hidden;
            transition: max-height 0.3s ease;
        }

        .collapsible-content.collapsed {
            max-height: 0;
        }

        .collapsible-body {
            padding: 15px;
        }

        .info-item {
            margin-bottom: 15px;
            padding: 15px;
            background: #f5f5f5;
            border-radius: 10px;
            border-left: 4px solid #2196F3;
        }

        .info-item h3 {
            margin: 0 0 5px 0;
            color: #2196F3;
            font-size: 0.9em;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .info-item p {
            margin: 0;
            color: #666;
            font-size: 0.95em;
        }

        .info-item input, .info-item textarea {
            width: 100%;
            padding: 8px;
            border: 1px solid #ddd;
            border-radius: 5px;
            font-family: inherit;
            font-size: 0.95em;
            margin-top: 5px;
        }

        .info-item textarea {
            resize: vertical;
            min-height: 60px;
        }

        .legend {
            position: absolute;
            bottom: 20px;
            left: 20px;
            background: rgba(255, 255, 255, 0.9);
            padding: 15px;
            border-radius: 10px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
        }

        .legend-item {
            display: flex;
            align-items: center;
            margin-bottom: 8px;
            font-size: 0.9em;
        }

        .legend-arrow {
            width: 30px;
            height: 2px;
            background: #666;
            margin-right: 10px;
            position: relative;
        }

        .legend-arrow::after {
            content: '';
            position: absolute;
            right: 0;
            top: -3px;
            width: 0;
            height: 0;
            border-left: 8px solid #666;
            border-top: 4px solid transparent;
            border-bottom: 4px solid transparent;
        }

        svg {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
        }

        svg.connecting {
            pointer-events: all;
        }

        .metrics {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            margin-top: 20px;
        }

        .metric {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 15px;
            border-radius: 10px;
            text-align: center;
        }

        .metric-value {
            font-size: 2em;
            font-weight: bold;
            margin-bottom: 5px;
        }

        .metric-label {
            font-size: 0.8em;
            opacity: 0.9;
        }

        .save-btn {
            background: #4CAF50;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 5px;
            cursor: pointer;
            width: 100%;
            margin-top: 10px;
            font-size: 16px;
        }

        .save-btn:hover {
            background: #45a049;
        }

        .modal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.5);
            align-items: center;
            justify-content: center;
            z-index: 1000;
        }

        .modal-content {
            background: white;
            padding: 30px;
            border-radius: 20px;
            max-width: 400px;
            width: 90%;
        }

        .modal h3 {
            margin-top: 0;
            color: #333;
        }

        .modal input, .modal select {
            width: 100%;
            padding: 10px;
            margin: 10px 0;
            border: 1px solid #ddd;
            border-radius: 5px;
            font-size: 16px;
        }

        .modal-buttons {
            display: flex;
            gap: 10px;
            margin-top: 20px;
        }

        .modal-buttons button {
            flex: 1;
            padding: 10px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
        }

        .modal-buttons .confirm {
            background: #2196F3;
            color: white;
        }

        .modal-buttons .cancel {
            background: #f5f5f5;
            color: #333;
        }

        .connection-line {
            stroke: #2196F3;
            stroke-width: 3;
            fill: none;
            opacity: 0.5;
            stroke-dasharray: 5,5;
        }

        .export-import {
            margin-top: 20px;
            display: flex;
            gap: 10px;
        }

        .export-import button {
            flex: 1;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 5px;
            background: white;
            cursor: pointer;
            font-size: 14px;
        }

        .export-import button:hover {
            background: #f5f5f5;
        }

        .persistence-controls {
            margin-top: 20px;
            padding: 15px;
            background: #f8f9fa;
            border-radius: 10px;
        }

        .persistence-controls button {
            width: 100%;
            padding: 10px;
            margin: 5px 0;
            border: 1px solid #ddd;
            border-radius: 5px;
            background: white;
            cursor: pointer;
            font-size: 14px;
        }

        .persistence-controls button:hover {
            background: #f5f5f5;
        }

        .persistence-controls button.danger {
            color: #f44336;
            border-color: #f44336;
        }

        .persistence-controls button.danger:hover {
            background: #ffebee;
        }
    </style>
</head>
<body>
    <div class="dashboard-header">
        <h1>Editable System Architecture Dashboard</h1>
        <div class="toolbar">
            <button id="add-component-btn">➕ Add Component</button>
            <button id="add-connection-btn">🔗 Add Connection</button>
            <button id="edit-mode-btn">✏️ Edit Mode</button>
            <button id="delete-mode-btn">🗑️ Delete Mode</button>
        </div>
    </div>
    
    <div class="main-container">
        <div class="diagram-container" id="diagram-container">
            <svg id="connections"></svg>
            
            <!-- Initial Components - Will be replaced by saved data if exists -->
            <div class="component" id="gp" style="left: 50px; top: 150px;">G.P</div>
            <div class="component" id="investor" style="left: 200px; top: 100px;">INVESTOR<br>Limited Partner</div>
            <div class="component" id="haki" style="left: 400px; top: 100px;">#AKI</div>
            <div class="component" id="sauray" style="left: 600px; top: 100px;">SAURAY</div>
            <div class="component" id="united-partnership" style="left: 150px; top: 250px;">UNITED<br>PARTNERSHIP</div>
            <div class="component" id="kaumutua" style="left: 400px; top: 300px;">KAUMUTUA<br>ENERGY</div>
            <div class="component" id="k-e-mgmt" style="left: 600px; top: 300px;">K E<br>MGMT (JT)</div>
            
            <div class="legend">
                <div class="legend-item">
                    <div class="legend-arrow"></div>
                    <span>Data/Resource Flow</span>
                </div>
                <div class="legend-item" style="opacity: 0.6;">
                    <div class="legend-arrow" style="border-style: dashed;"></div>
                    <span>Partnership/Relationship</span>
                </div>
            </div>
        </div>
        
        <button class="panel-toggle" id="panel-toggle" onclick="toggleSidePanel()">◀</button>
        
        <div class="info-panel" id="info-panel">
            <div class="collapsible-section">
                <div class="collapsible-header" onclick="toggleComponentDetails(this)" style="margin: -20px -20px 20px -20px; padding: 20px; background: #f8f9fa; border-radius: 20px 20px 0 0;">
                    <h2 style="margin: 0; color: #333;">Component Details</h2>
                    <span class="collapse-icon">▼</span>
                </div>
            </div>
            
            <div class="component-details-wrapper" id="component-details-wrapper">
                <div id="component-info">
                    <div class="info-item">
                        <h3>Select a Component</h3>
                        <p>Click on any component to view and edit its details. Use the sections below to explore different aspects of each component.</p>
                    </div>
                </div>
            </div>
            
            <div class="collapsible-section" style="margin-top: 20px;">
                <div class="collapsible-header" onclick="toggleCollapse(this)">
                    <h3>Dashboard Metrics</h3>
                    <span class="collapse-icon">▼</span>
                </div>
                <div class="collapsible-content">
                    <div class="collapsible-body" style="padding: 10px;">
                        <div class="metrics">
                            <div class="metric">
                                <div class="metric-value" id="total-components">7</div>
                                <div class="metric-label">Total Components</div>
                            </div>
                            <div class="metric">
                                <div class="metric-value" id="total-connections">7</div>
                                <div class="metric-label">Connections</div>
                            </div>
                            <div class="metric">
                                <div class="metric-value" id="active-flows">5</div>
                                <div class="metric-label">Active Flows</div>
                            </div>
                            <div class="metric">
                                <div class="metric-value" id="partnerships">2</div>
                                <div class="metric-label">Partnerships</div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <div class="export-import">
                <button id="export-btn">📥 Export</button>
                <button id="import-btn">📤 Import</button>
            </div>

            <div class="persistence-controls">
                <h3 style="margin: 0 0 10px 0; font-size: 1em; color: #666;">Persistence Options</h3>
                <button id="save-manually-btn">💾 Save to Browser</button>
                <button id="reset-to-default-btn" class="danger">🔄 Reset to Default</button>
            </div>
        </div>
    </div>

    <!-- Add Component Modal -->
    <div class="modal" id="add-component-modal">
        <div class="modal-content">
            <h3>Add New Component</h3>
            <input type="text" id="new-component-name" placeholder="Component Name">
            <select id="new-component-type">
                <option value="Management">Management</option>
                <option value="Investment">Investment</option>
                <option value="Operations">Operations</option>
                <option value="Energy">Energy</option>
                <option value="Partnership">Partnership</option>
                <option value="Technology">Technology</option>
            </select>
            <input type="text" id="new-component-description" placeholder="Description">
            <div class="modal-buttons">
                <button class="cancel" onclick="closeModal('add-component-modal')">Cancel</button>
                <button class="confirm" onclick="addNewComponent()">Add Component</button>
            </div>
        </div>
    </div>

    <!-- Add Connection Modal -->
    <div class="modal" id="add-connection-modal">
        <div class="modal-content">
            <h3>Add New Connection</h3>
            <select id="connection-from">
                <option value="">Select Source Component</option>
            </select>
            <select id="connection-to">
                <option value="">Select Target Component</option>
            </select>
            <select id="connection-type">
                <option value="solid">Solid (Direct Flow)</option>
                <option value="dashed">Dashed (Partnership)</option>
            </select>
            <div class="modal-buttons">
                <button class="cancel" onclick="closeModal('add-connection-modal')">Cancel</button>
                <button class="confirm" onclick="addNewConnection()">Add Connection</button>
            </div>
        </div>
    </div>

    <script>
        // Storage key for localStorage
        const STORAGE_KEY = 'architecture-dashboard-data';

        // Default component data
        const defaultComponentData = {
            'gp': {
                name: 'G.P (General Partner)',
                description: 'The General Partner manages the fund and makes investment decisions.',
                type: 'Management'
            },
            'investor': {
                name: 'INVESTOR - Limited Partner',
                description: 'Limited Partners provide capital to the fund but have limited liability.',
                type: 'Investment'
            },
            'haki': {
                name: '#AKI',
                description: 'Strategic component in the system architecture.',
                type: 'Operations'
            },
            'sauray': {
                name: 'SAURAY',
                description: 'Key operational unit with direct management connections.',
                type: 'Operations'
            },
            'united-partnership': {
                name: 'UNITED PARTNERSHIP',
                description: 'Central partnership entity managing various stakeholders.',
                type: 'Partnership'
            },
            'kaumutua': {
                name: 'KAUMUTUA ENERGY',
                description: 'Energy division handling power and resource management.',
                type: 'Energy'
            },
            'k-e-mgmt': {
                name: 'K E MGMT (JT)',
                description: 'Joint management entity coordinating operations.',
                type: 'Management'
            }
        };

        // Default connection definitions
        const defaultConnections = [
            { from: 'gp', to: 'united-partnership', type: 'solid' },
            { from: 'investor', to: 'united-partnership', type: 'solid' },
            { from: 'united-partnership', to: 'haki', type: 'dashed' },
            { from: 'united-partnership', to: 'sauray', type: 'dashed' },
            { from: 'haki', to: 'kaumutua', type: 'solid' },
            { from: 'sauray', to: 'k-e-mgmt', type: 'solid' },
            { from: 'kaumutua', to: 'k-e-mgmt', type: 'solid', bidirectional: true }
        ];

        // Component data - now mutable
        let componentData = {...defaultComponentData};

        // Connection definitions - now mutable
        let connections = [...defaultConnections];

        let selectedComponent = null;
        let editMode = false;
        let deleteMode = false;
        let connectingMode = false;
        let connectionStart = null;
        let draggedElement = null;
        let offset = { x: 0, y: 0 };

        function drawConnections() {
            const svg = document.getElementById('connections');
            svg.innerHTML = '';

            // Define arrow markers
            const defs = document.createElementNS('http://www.w3.org/2000/svg', 'defs');
            
            // Regular arrow marker
            const marker = document.createElementNS('http://www.w3.org/2000/svg', 'marker');
            marker.setAttribute('id', 'arrowhead');
            marker.setAttribute('markerWidth', '15');
            marker.setAttribute('markerHeight', '15');
            marker.setAttribute('refX', '14');
            marker.setAttribute('refY', '7.5');
            marker.setAttribute('orient', 'auto');
            marker.setAttribute('markerUnits', 'strokeWidth');
            
            const polygon = document.createElementNS('http://www.w3.org/2000/svg', 'polygon');
            polygon.setAttribute('points', '0 0, 15 7.5, 0 15');
            polygon.setAttribute('class', 'arrow-marker');
            
            marker.appendChild(polygon);
            defs.appendChild(marker);
            
            // Highlighted arrow marker
            const markerHighlighted = document.createElementNS('http://www.w3.org/2000/svg', 'marker');
            markerHighlighted.setAttribute('id', 'arrowhead-highlighted');
            markerHighlighted.setAttribute('markerWidth', '15');
            markerHighlighted.setAttribute('markerHeight', '15');
            markerHighlighted.setAttribute('refX', '14');
            markerHighlighted.setAttribute('refY', '7.5');
            markerHighlighted.setAttribute('orient', 'auto');
            markerHighlighted.setAttribute('markerUnits', 'strokeWidth');
            
            const polygonHighlighted = document.createElementNS('http://www.w3.org/2000/svg', 'polygon');
            polygonHighlighted.setAttribute('points', '0 0, 15 7.5, 0 15');
            polygonHighlighted.setAttribute('class', 'arrow-marker-highlighted');
            
            markerHighlighted.appendChild(polygonHighlighted);
            defs.appendChild(markerHighlighted);
            
            svg.appendChild(defs);

            connections.forEach((conn, index) => {
                const fromEl = document.getElementById(conn.from);
                const toEl = document.getElementById(conn.to);

                if (fromEl && toEl) {
                    const fromRect = fromEl.getBoundingClientRect();
                    const toRect = toEl.getBoundingClientRect();
                    const containerRect = svg.parentElement.getBoundingClientRect();

                    const fromX = fromRect.left + fromRect.width / 2 - containerRect.left;
                    const fromY = fromRect.top + fromRect.height / 2 - containerRect.top;
                    const toX = toRect.left + toRect.width / 2 - containerRect.left;
                    const toY = toRect.top + toRect.height / 2 - containerRect.top;

                    // Calculate edge points instead of center points for better arrow positioning
                    const angle = Math.atan2(toY - fromY, toX - fromX);
                    
                    // Adjust start and end points to component edges
                    const fromEdgeX = fromX + Math.cos(angle) * (fromRect.width / 2);
                    const fromEdgeY = fromY + Math.sin(angle) * (fromRect.height / 2);
                    const toEdgeX = toX - Math.cos(angle) * (toRect.width / 2 + 15); // Extra space for arrow
                    const toEdgeY = toY - Math.sin(angle) * (toRect.height / 2 + 15);

                    const path = document.createElementNS('http://www.w3.org/2000/svg', 'path');
                    
                    // Create curved path with adjusted points
                    const midX = (fromEdgeX + toEdgeX) / 2;
                    const midY = (fromEdgeY + toEdgeY) / 2;
                    const curvature = 30;
                    
                    const d = `M ${fromEdgeX} ${fromEdgeY} Q ${midX} ${midY - curvature} ${toEdgeX} ${toEdgeY}`;
                    path.setAttribute('d', d);
                    path.classList.add('connection');
                    path.setAttribute('id', `conn-${index}`);
                    path.setAttribute('data-from', conn.from);
                    path.setAttribute('data-to', conn.to);
                    path.setAttribute('data-index', index);
                    
                    if (conn.type === 'dashed') {
                        path.setAttribute('stroke-dasharray', '8,5');
                    }
                    
                    path.setAttribute('marker-end', 'url(#arrowhead)');
                    
                    // Add bidirectional arrow if needed
                    if (conn.bidirectional) {
                        path.setAttribute('marker-start', 'url(#arrowhead)');
                    }
                    
                    // Add hover effect
                    path.addEventListener('mouseenter', function() {
                        this.setAttribute('marker-end', 'url(#arrowhead-highlighted)');
                        if (conn.bidirectional) {
                            this.setAttribute('marker-start', 'url(#arrowhead-highlighted)');
                        }
                    });
                    
                    path.addEventListener('mouseleave', function() {
                        if (!this.classList.contains('highlighted')) {
                            this.setAttribute('marker-end', 'url(#arrowhead)');
                            if (conn.bidirectional) {
                                this.setAttribute('marker-start', 'url(#arrowhead)');
                            }
                        }
                    });
                    
                    // Add click handler for deletion
                    path.addEventListener('click', function(e) {
                        if (deleteMode) {
                            e.stopPropagation();
                            if (confirm('Delete this connection?')) {
                                connections.splice(index, 1);
                                drawConnections();
                                updateMetrics();
                                triggerAutoSave();
                            }
                        }
                    });
                    
                    svg.appendChild(path);
                }
            });
        }

        function highlightConnections(componentId) {
            document.querySelectorAll('.connection').forEach(conn => {
                conn.classList.remove('highlighted');
                conn.setAttribute('marker-end', 'url(#arrowhead)');
                conn.setAttribute('marker-start', '');
            });
            document.querySelectorAll('.arrow-marker').forEach(arrow => {
                arrow.style.fill = '#666';
            });

            document.querySelectorAll('.connection').forEach(conn => {
                if (conn.getAttribute('data-from') === componentId || 
                    conn.getAttribute('data-to') === componentId) {
                    conn.classList.add('highlighted');
                    conn.setAttribute('marker-end', 'url(#arrowhead-highlighted)');
                    
                    // Check if bidirectional
                    const index = parseInt(conn.getAttribute('data-index'));
                    if (connections[index] && connections[index].bidirectional) {
                        conn.setAttribute('marker-start', 'url(#arrowhead-highlighted)');
                    }
                }
            });
        }

        function updateInfoPanel(componentId) {
            const data = componentData[componentId];
            const infoDiv = document.getElementById('component-info');
            
            // Get connected components
            const outgoingConnections = connections
                .filter(conn => conn.from === componentId)
                .map(conn => componentData[conn.to]?.name || conn.to);
            
            const incomingConnections = connections
                .filter(conn => conn.to === componentId)
                .map(conn => componentData[conn.from]?.name || conn.from);
            
            infoDiv.innerHTML = `
                <div class="collapsible-section">
                    <div class="collapsible-header" onclick="toggleCollapse(this)">
                        <h3>Basic Information</h3>
                        <span class="collapse-icon">▼</span>
                    </div>
                    <div class="collapsible-content">
                        <div class="collapsible-body">
                            <div class="info-item">
                                <h3>Component Name</h3>
                                <input type="text" id="edit-name" value="${data.name}" ${!editMode ? 'disabled' : ''}>
                            </div>
                            <div class="info-item">
                                <h3>Type</h3>
                                <select id="edit-type" ${!editMode ? 'disabled' : ''}>
                                    <option value="Management" ${data.type === 'Management' ? 'selected' : ''}>Management</option>
                                    <option value="Investment" ${data.type === 'Investment' ? 'selected' : ''}>Investment</option>
                                    <option value="Operations" ${data.type === 'Operations' ? 'selected' : ''}>Operations</option>
                                    <option value="Energy" ${data.type === 'Energy' ? 'selected' : ''}>Energy</option>
                                    <option value="Partnership" ${data.type === 'Partnership' ? 'selected' : ''}>Partnership</option>
                                    <option value="Technology" ${data.type === 'Technology' ? 'selected' : ''}>Technology</option>
                                </select>
                            </div>
                        </div>
                    </div>
                </div>
                
                <div class="collapsible-section">
                    <div class="collapsible-header" onclick="toggleCollapse(this)">
                        <h3>Description</h3>
                        <span class="collapse-icon">▼</span>
                    </div>
                    <div class="collapsible-content">
                        <div class="collapsible-body">
                            <div class="info-item">
                                <textarea id="edit-description" ${!editMode ? 'disabled' : ''}>${data.description}</textarea>
                            </div>
                        </div>
                    </div>
                </div>
                
                <div class="collapsible-section">
                    <div class="collapsible-header" onclick="toggleCollapse(this)">
                        <h3>Connections</h3>
                        <span class="collapse-icon">▼</span>
                    </div>
                    <div class="collapsible-content">
                        <div class="collapsible-body">
                            <div class="info-item">
                                <h3>Outgoing Connections (${outgoingConnections.length})</h3>
                                <p>${outgoingConnections.length > 0 ? outgoingConnections.join(', ') : 'None'}</p>
                            </div>
                            <div class="info-item">
                                <h3>Incoming Connections (${incomingConnections.length})</h3>
                                <p>${incomingConnections.length > 0 ? incomingConnections.join(', ') : 'None'}</p>
                            </div>
                        </div>
                    </div>
                </div>
                
                ${editMode ? '<button class="save-btn" onclick="saveComponentChanges()">Save Changes</button>' : ''}
            `;
            
            // Restore any previously collapsed states for component subsections
            if (window.collapsedSections && window.collapsedSections[componentId]) {
                // Skip the main Component Details header (index 0) when restoring subsection states
                const subsectionHeaders = document.querySelectorAll('#component-info .collapsible-header');
                Object.entries(window.collapsedSections[componentId]).forEach(([index, isCollapsed]) => {
                    if (isCollapsed && subsectionHeaders[index]) {
                        toggleCollapse(subsectionHeaders[index]);
                    }
                });
            }
        }

        // Toggle component details section
        function toggleComponentDetails(header) {
            const wrapper = document.getElementById('component-details-wrapper');
            const icon = header.querySelector('.collapse-icon');
            const isCollapsed = wrapper.classList.contains('collapsed');
            
            if (isCollapsed) {
                wrapper.classList.remove('collapsed');
                header.classList.remove('collapsed');
                icon.textContent = '▼';
            } else {
                wrapper.classList.add('collapsed');
                header.classList.add('collapsed');
                icon.textContent = '▶';
            }
            
            window.componentDetailsCollapsed = !isCollapsed;
            triggerAutoSave();
        }

        // Toggle other collapsible sections
        function toggleCollapse(header) {
            const content = header.nextElementSibling;
            const isCollapsed = content.classList.contains('collapsed');
            
            if (isCollapsed) {
                content.classList.remove('collapsed');
                header.classList.remove('collapsed');
            } else {
                content.classList.add('collapsed');
                header.classList.add('collapsed');
            }
            
            // Special handling for main sections
            const headerText = header.querySelector('h2, h3').textContent;
            
            if (headerText === 'Dashboard Metrics') {
                window.metricsCollapsed = !isCollapsed;
            } else if (selectedComponent) {
                // Save collapse state for component subsections
                if (!window.collapsedSections) {
                    window.collapsedSections = {};
                }
                if (!window.collapsedSections[selectedComponent]) {
                    window.collapsedSections[selectedComponent] = {};
                }
                
                const subsectionHeaders = Array.from(document.querySelectorAll('#component-info .collapsible-header'));
                const index = subsectionHeaders.indexOf(header);
                if (index !== -1) {
                    window.collapsedSections[selectedComponent][index] = !isCollapsed;
                }
            }
            
            triggerAutoSave();
        }

        function saveComponentChanges() {
            if (selectedComponent) {
                const newName = document.getElementById('edit-name').value;
                const newType = document.getElementById('edit-type').value;
                const newDescription = document.getElementById('edit-description').value;
                
                componentData[selectedComponent].name = newName;
                componentData[selectedComponent].type = newType;
                componentData[selectedComponent].description = newDescription;
                
                // Update the visual component
                const componentEl = document.getElementById(selectedComponent);
                componentEl.innerHTML = newName.includes('<br>') ? newName : newName.replace(/\s-\s/g, '<br>');
                
                // Re-add delete button
                const deleteBtn = componentEl.querySelector('.delete-btn');
                if (deleteBtn) {
                    componentEl.appendChild(deleteBtn);
                }
                
                alert('Component updated successfully!');
                triggerAutoSave();
            }
        }

        function setupDragAndDrop(element) {
            element.addEventListener('mousedown', startDrag);
        }

        function startDrag(e) {
            if (deleteMode || connectingMode) return;
            
            draggedElement = e.target;
            const rect = draggedElement.getBoundingClientRect();
            const containerRect = draggedElement.parentElement.getBoundingClientRect();
            
            offset.x = e.clientX - rect.left;
            offset.y = e.clientY - rect.top;
            
            document.addEventListener('mousemove', drag);
            document.addEventListener('mouseup', stopDrag);
        }

        function drag(e) {
            if (!draggedElement) return;
            
            const containerRect = draggedElement.parentElement.getBoundingClientRect();
            const x = e.clientX - containerRect.left - offset.x;
            const y = e.clientY - containerRect.top - offset.y;
            
            draggedElement.style.left = `${x}px`;
            draggedElement.style.top = `${y}px`;
            
            drawConnections();
        }

        function stopDrag() {
            if (draggedElement) {
                triggerAutoSave();
            }
            draggedElement = null;
            document.removeEventListener('mousemove', drag);
            document.removeEventListener('mouseup', stopDrag);
        }

        function initializeComponent(component) {
            setupDragAndDrop(component);
            
            // Add delete button
            const deleteBtn = document.createElement('button');
            deleteBtn.className = 'delete-btn';
            deleteBtn.innerHTML = '×';
            deleteBtn.onclick = function(e) {
                e.stopPropagation();
                if (confirm('Delete this component and all its connections?')) {
                    const componentId = component.id;
                    delete componentData[componentId];
                    connections = connections.filter(conn => 
                        conn.from !== componentId && conn.to !== componentId
                    );
                    component.remove();
                    drawConnections();
                    updateMetrics();
                    triggerAutoSave();
                }
            };
            component.appendChild(deleteBtn);
            
            component.addEventListener('click', function(e) {
                if (deleteMode || draggedElement) return;
                
                if (connectingMode) {
                    if (!connectionStart) {
                        connectionStart = this.id;
                        this.style.border = '3px solid #2196F3';
                    } else if (connectionStart !== this.id) {
                        connections.push({
                            from: connectionStart,
                            to: this.id,
                            type: 'solid'
                        });
                        drawConnections();
                        updateMetrics();
                        
                        // Reset
                        document.getElementById(connectionStart).style.border = '';
                        connectionStart = null;
                        connectingMode = false;
                        document.getElementById('add-connection-btn').classList.remove('active');
                        document.getElementById('diagram-container').classList.remove('connecting');
                    }
                    return;
                }
                
                document.querySelectorAll('.component').forEach(c => c.classList.remove('active'));
                this.classList.add('active');
                selectedComponent = this.id;
                highlightConnections(this.id);
                updateInfoPanel(this.id);
            });
            
            component.addEventListener('dblclick', function() {
                if (!editMode) return;
                
                const currentText = this.innerText;
                const input = document.createElement('input');
                input.type = 'text';
                input.value = currentText;
                input.style.width = '100%';
                input.style.fontSize = 'inherit';
                input.style.fontWeight = 'inherit';
                input.style.border = 'none';
                input.style.background = 'transparent';
                input.style.textAlign = 'center';
                
                this.innerHTML = '';
                this.appendChild(input);
                input.focus();
                input.select();
                
                const saveText = () => {
                    const newText = input.value;
                    this.innerHTML = newText;
                    if (componentData[this.id]) {
                        componentData[this.id].name = newText;
                    }
                    this.appendChild(deleteBtn);
                    triggerAutoSave();
                };
                
                input.addEventListener('blur', saveText);
                input.addEventListener('keypress', function(e) {
                    if (e.key === 'Enter') {
                        saveText();
                    }
                });
            });
        }

        function addNewComponent() {
            const name = document.getElementById('new-component-name').value;
            const type = document.getElementById('new-component-type').value;
            const description = document.getElementById('new-component-description').value;
            
            if (!name) {
                alert('Please enter a component name');
                return;
            }
            
            const id = name.toLowerCase().replace(/\s+/g, '-') + '-' + Date.now();
            
            componentData[id] = {
                name: name,
                type: type,
                description: description
            };
            
            const component = document.createElement('div');
            component.className = 'component';
            component.id = id;
            component.style.left = '50px';
            component.style.top = '50px';
            component.innerHTML = name;
            
            document.getElementById('diagram-container').appendChild(component);
            initializeComponent(component);
            
            closeModal('add-component-modal');
            updateMetrics();
            triggerAutoSave();
            
            // Clear form
            document.getElementById('new-component-name').value = '';
            document.getElementById('new-component-description').value = '';
        }

        function addNewConnection() {
            const from = document.getElementById('connection-from').value;
            const to = document.getElementById('connection-to').value;
            const type = document.getElementById('connection-type').value;
            
            if (!from || !to) {
                alert('Please select both source and target components');
                return;
            }
            
            if (from === to) {
                alert('Cannot connect a component to itself');
                return;
            }
            
            connections.push({
                from: from,
                to: to,
                type: type
            });
            
            drawConnections();
            closeModal('add-connection-modal');
            updateMetrics();
            triggerAutoSave();
        }

        function updateMetrics() {
            document.getElementById('total-components').textContent = Object.keys(componentData).length;
            document.getElementById('total-connections').textContent = connections.length;
            document.getElementById('active-flows').textContent = connections.filter(c => c.type === 'solid').length;
            document.getElementById('partnerships').textContent = connections.filter(c => c.type === 'dashed').length;
        }

        function closeModal(modalId) {
            document.getElementById(modalId).style.display = 'none';
        }

        // Toggle side panel function
        function toggleSidePanel() {
            const panel = document.getElementById('info-panel');
            const toggle = document.getElementById('panel-toggle');
            const diagram = document.getElementById('diagram-container');
            
            if (panel.classList.contains('collapsed')) {
                panel.classList.remove('collapsed');
                toggle.classList.remove('panel-collapsed');
                toggle.innerHTML = '◀';
                diagram.classList.remove('expanded');
                window.sidePanelCollapsed = false;
            } else {
                panel.classList.add('collapsed');
                toggle.classList.add('panel-collapsed');
                toggle.innerHTML = '▶';
                diagram.classList.add('expanded');
                window.sidePanelCollapsed = true;
            }
            
            // Redraw connections after animation
            setTimeout(() => {
                drawConnections();
            }, 300);
            
            triggerAutoSave();
        }

        // Save and Load functions for true persistence
        function saveToLocalStorage() {
            const components = {};
            document.querySelectorAll('.component').forEach(comp => {
                components[comp.id] = {
                    ...componentData[comp.id],
                    position: {
                        left: comp.style.left,
                        top: comp.style.top
                    }
                };
            });
            
            const data = {
                components: components,
                connections: connections,
                lastSaved: new Date().toISOString(),
                sidePanelCollapsed: window.sidePanelCollapsed || false,
                metricsCollapsed: window.metricsCollapsed || false,
                componentDetailsCollapsed: window.componentDetailsCollapsed || false,
                collapsedSections: window.collapsedSections || {}
            };
            
            // Save to localStorage
            try {
                localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
                
                // Show save indicator
                const indicator = document.createElement('div');
                indicator.style.position = 'fixed';
                indicator.style.top = '20px';
                indicator.style.right = '20px';
                indicator.style.background = '#4CAF50';
                indicator.style.color = 'white';
                indicator.style.padding = '10px 20px';
                indicator.style.borderRadius = '5px';
                indicator.style.boxShadow = '0 2px 5px rgba(0,0,0,0.2)';
                indicator.textContent = '✓ Auto-saved';
                indicator.style.zIndex = '1001';
                document.body.appendChild(indicator);
                
                setTimeout(() => {
                    indicator.remove();
                }, 2000);
            } catch (err) {
                console.error('Error saving to localStorage:', err);
            }
        }

        function loadFromLocalStorage() {
            try {
                // Try to load from localStorage
                const storedData = localStorage.getItem(STORAGE_KEY);
                if (!storedData) return false;
                
                const data = JSON.parse(storedData);
                
                // Clear existing components
                document.querySelectorAll('.component').forEach(c => c.remove());
                
                // Load components
                componentData = {};
                for (const [id, compData] of Object.entries(data.components)) {
                    componentData[id] = {
                        name: compData.name,
                        type: compData.type,
                        description: compData.description
                    };
                    
                    const component = document.createElement('div');
                    component.className = 'component';
                    component.id = id;
                    component.style.left = compData.position.left;
                    component.style.top = compData.position.top;
                    component.innerHTML = compData.name;
                    
                    document.getElementById('diagram-container').appendChild(component);
                    initializeComponent(component);
                }
                
                // Load connections
                connections = data.connections || [];
                
                // Restore UI states
                window.sidePanelCollapsed = data.sidePanelCollapsed || false;
                window.metricsCollapsed = data.metricsCollapsed || false;
                window.componentDetailsCollapsed = data.componentDetailsCollapsed || false;
                window.collapsedSections = data.collapsedSections || {};
                
                // Apply side panel state
                if (window.sidePanelCollapsed) {
                    toggleSidePanel();
                }
                
                drawConnections();
                updateMetrics();
                
                return true;
            } catch (err) {
                console.error('Error loading saved data:', err);
                return false;
            }
        }

        function resetToDefault() {
            if (confirm('Are you sure you want to reset to the default diagram? This will clear all your changes.')) {
                // Clear localStorage
                localStorage.removeItem(STORAGE_KEY);
                
                // Reset to defaults
                componentData = {...defaultComponentData};
                connections = [...defaultConnections];
                
                // Clear existing components
                document.querySelectorAll('.component').forEach(c => c.remove());
                
                // Recreate default components
                const defaultPositions = {
                    'gp': { left: '50px', top: '150px' },
                    'investor': { left: '200px', top: '100px' },
                    'haki': { left: '400px', top: '100px' },
                    'sauray': { left: '600px', top: '100px' },
                    'united-partnership': { left: '150px', top: '250px' },
                    'kaumutua': { left: '400px', top: '300px' },
                    'k-e-mgmt': { left: '600px', top: '300px' }
                };
                
                for (const [id, compData] of Object.entries(defaultComponentData)) {
                    const component = document.createElement('div');
                    component.className = 'component';
                    component.id = id;
                    component.style.left = defaultPositions[id].left;
                    component.style.top = defaultPositions[id].top;
                    component.innerHTML = compData.name.replace(/\s-\s/g, '<br>');
                    
                    document.getElementById('diagram-container').appendChild(component);
                    initializeComponent(component);
                }
                
                // Reset UI states
                window.sidePanelCollapsed = false;
                window.metricsCollapsed = false;
                window.componentDetailsCollapsed = false;
                window.collapsedSections = {};
                
                if (document.getElementById('info-panel').classList.contains('collapsed')) {
                    toggleSidePanel();
                }
                
                drawConnections();
                updateMetrics();
                
                alert('Dashboard reset to default state.');
            }
        }

        // Auto-save functionality
        let saveTimeout;
        function triggerAutoSave() {
            clearTimeout(saveTimeout);
            saveTimeout = setTimeout(() => {
                saveToLocalStorage();
            }, 1000); // Save 1 second after last change
        }

        // Export function
        function exportDiagram() {
            const components = {};
            document.querySelectorAll('.component').forEach(comp => {
                components[comp.id] = {
                    ...componentData[comp.id],
                    position: {
                        left: comp.style.left,
                        top: comp.style.top
                    }
                };
            });
            
            const data = {
                components: components,
                connections: connections
            };
            
            const blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = 'architecture-diagram.json';
            a.click();
        }

        // Import function
        function importDiagram() {
            const input = document.createElement('input');
            input.type = 'file';
            input.accept = '.json';
            input.onchange = function(e) {
                const file = e.target.files[0];
                const reader = new FileReader();
                reader.onload = function(e) {
                    try {
                        const data = JSON.parse(e.target.result);
                        
                        // Clear existing components
                        document.querySelectorAll('.component').forEach(c => c.remove());
                        
                        // Load imported components
                        componentData = {};
                        for (const [id, compData] of Object.entries(data.components)) {
                            componentData[id] = {
                                name: compData.name,
                                type: compData.type,
                                description: compData.description
                            };
                            
                            const component = document.createElement('div');
                            component.className = 'component';
                            component.id = id;
                            component.style.left = compData.position.left;
                            component.style.top = compData.position.top;
                            component.innerHTML = compData.name;
                            
                            document.getElementById('diagram-container').appendChild(component);
                            initializeComponent(component);
                        }
                        
                        // Load connections
                        connections = data.connections || [];
                        
                        drawConnections();
                        updateMetrics();
                        triggerAutoSave();
                        
                        alert('Diagram imported successfully!');
                    } catch (err) {
                        alert('Error importing file: ' + err.message);
                    }
                };
                reader.readAsText(file);
            };
            input.click();
        }

        // Initialize and load saved state
        window.addEventListener('load', () => {
            // Try to load saved diagram
            const loaded = loadFromLocalStorage();
            
            if (!loaded) {
                // Initialize with default components if no saved data
                document.querySelectorAll('.component').forEach(component => {
                    initializeComponent(component);
                });
            }
            
            drawConnections();
            updateMetrics();
            
            // Restore collapsed states after a short delay to ensure DOM is ready
            setTimeout(() => {
                // Restore Component Details collapsed state
                if (window.componentDetailsCollapsed) {
                    const componentDetailsHeader = document.querySelector('.collapsible-header[onclick*="toggleComponentDetails"]');
                    if (componentDetailsHeader) {
                        toggleComponentDetails(componentDetailsHeader);
                    }
                }
                
                // Restore metrics collapsed state
                if (window.metricsCollapsed) {
                    const metricsHeader = Array.from(document.querySelectorAll('.collapsible-header'))
                        .find(h => h.querySelector('h3') && h.querySelector('h3').textContent === 'Dashboard Metrics');
                    if (metricsHeader) {
                        toggleCollapse(metricsHeader);
                    }
                }
            }, 100);
        });

        // Event listeners
        document.getElementById('add-component-btn').addEventListener('click', function() {
            document.getElementById('add-component-modal').style.display = 'flex';
        });

        document.getElementById('add-connection-btn').addEventListener('click', function() {
            if (connectingMode) {
                connectingMode = false;
                this.classList.remove('active');
                document.getElementById('diagram-container').classList.remove('connecting');
                if (connectionStart) {
                    document.getElementById(connectionStart).style.border = '';
                    connectionStart = null;
                }
            } else {
                // Show modal with component options
                const fromSelect = document.getElementById('connection-from');
                const toSelect = document.getElementById('connection-to');
                
                fromSelect.innerHTML = '<option value="">Select Source Component</option>';
                toSelect.innerHTML = '<option value="">Select Target Component</option>';
                
                Object.entries(componentData).forEach(([id, data]) => {
                    fromSelect.innerHTML += `<option value="${id}">${data.name}</option>`;
                    toSelect.innerHTML += `<option value="${id}">${data.name}</option>`;
                });
                
                document.getElementById('add-connection-modal').style.display = 'flex';
            }
        });

        document.getElementById('edit-mode-btn').addEventListener('click', function() {
            editMode = !editMode;
            this.classList.toggle('active');
            if (selectedComponent) {
                updateInfoPanel(selectedComponent);
            }
        });

        document.getElementById('delete-mode-btn').addEventListener('click', function() {
            deleteMode = !deleteMode;
            this.classList.toggle('active');
            document.querySelectorAll('.component').forEach(comp => {
                comp.style.cursor = deleteMode ? 'not-allowed' : 'move';
            });
            document.querySelectorAll('.connection').forEach(conn => {
                conn.style.cursor = deleteMode ? 'pointer' : 'default';
            });
        });

        document.getElementById('export-btn').addEventListener('click', exportDiagram);
        document.getElementById('import-btn').addEventListener('click', importDiagram);
        document.getElementById('save-manually-btn').addEventListener('click', function() {
            saveToLocalStorage();
        });
        document.getElementById('reset-to-default-btn').addEventListener('click', resetToDefault);

        // Close modals when clicking outside
        document.querySelectorAll('.modal').forEach(modal => {
            modal.addEventListener('click', function(e) {
                if (e.target === this) {
                    this.style.display = 'none';
                }
            });
        });

        window.addEventListener('resize', drawConnections);
    </script>
</body>
</html>
