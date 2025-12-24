---
layout: post
title: "Happy Christmas"
date: 2025-12-24
categories: "Fun"
---
点击屏幕喔🎄🎅🎁❄️✨
<!-- <!DOCTYPE html> -->
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>Happy Christmas!</title>
    <style>
        body { margin: 0; overflow: hidden; background-color: #151028; }
        #container {
            max-width: 1000px;
            margin: 20px auto;
            position: relative;
            width: calc(100% - 40px);
            height: 600px;
            overflow: hidden;
            background-color: #250d63;
            border-radius: 8px;
            box-shadow: 0 0 30px rgba(0, 0, 0, 0.5);
        }
        #loading {
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            color: white; font-family: sans-serif; letter-spacing: 2px;
            z-index: 10;
        }
        #starfield {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            background-color: #250d63;
            pointer-events: none;
            z-index: 1;
        }
        canvas {
            position: absolute;
            top: 0;
            left: 0;
            z-index: 2;
        }
    </style>
</head>
<body>
    <div id="container">
        <div id="starfield"></div>
        <div id="loading">正在解析 3D 模型...</div>
    </div>

    <script src="https://unpkg.com/three@0.128.0/build/three.min.js"></script>
    <script src="https://unpkg.com/gsap@3.9.1/dist/gsap.min.js"></script>

    <script src="https://unpkg.com/three@0.128.0/examples/js/loaders/GLTFLoader.js"></script>
    <script src="https://unpkg.com/three@0.128.0/examples/js/math/MeshSurfaceSampler.js"></script>
    <script src="https://unpkg.com/three@0.128.0/examples/js/postprocessing/EffectComposer.js"></script>
    <script src="https://unpkg.com/three@0.128.0/examples/js/postprocessing/RenderPass.js"></script>
    <script src="https://unpkg.com/three@0.128.0/examples/js/postprocessing/ShaderPass.js"></script>
    <script src="https://unpkg.com/three@0.128.0/examples/js/postprocessing/UnrealBloomPass.js"></script>
    <script src="https://unpkg.com/three@0.128.0/examples/js/shaders/CopyShader.js"></script>
    <script src="https://unpkg.com/three@0.128.0/examples/js/shaders/LuminosityHighPassShader.js"></script>

    <script>
        let scene, camera, renderer, particles;
        const APPLE_PARTICLE_COUNT = 8000;      // 苹果粒子数
        const TREE_PARTICLE_COUNT = 8000;       // 圣诞树粒子数
        const CHRISTMAS_PARTICLE_COUNT = 8000;  // 文字粒子数
        let currentParticleCount = APPLE_PARTICLE_COUNT; // 当前粒子数
        let applePoints = [];
        let treePoints = [];
        let christmasPoints = [];
        let currentShape = 'apple';
        let composer;
        let particleColors = []; // 存储粒子颜色

        // 资源链接 (这里可以使用任何公开的 .glb 模型)
        // 注意：为了演示，如果模型加载失败，代码会自动回退到之前的数学解析模式
        const APPLE_MODEL_URL = '../../../../assets/Apple.glb'; // 苹果模型
        const TREE_MODEL_URL = '../../../../assets/tree.glb';   // 圣诞树模型

        init();

        // 创建星空背景
        function createStarfield() {
            const starfield = document.getElementById('starfield');
            const starCount = 200; // 星星数量
            
            for (let i = 0; i < starCount; i++) {
                const star = document.createElement('div');
                const x = Math.random() * 100;
                const y = Math.random() * 100;
                const size = Math.random() * 2 + 0.5; // 0.5-2.5px
                const duration = Math.random() * 2 + 1; // 1-3秒闪烁周期
                const delay = Math.random() * 5; // 0-5秒随机延迟
                
                // 随机颜色（白色、蓝色、黄色等）
                const colors = ['#fff', '#fff9e6', '#e6f2ff', '#ffe6e6', '#f0e6ff'];
                const color = colors[Math.floor(Math.random() * colors.length)];
                
                star.style.cssText = `
                    position: absolute;
                    left: ${x}%;
                    top: ${y}%;
                    width: ${size}px;
                    height: ${size}px;
                    background-color: ${color};
                    border-radius: 50%;
                    box-shadow: 0 0 ${size * 2}px ${color};
                    animation: twinkle ${duration}s infinite;
                    animation-delay: ${delay}s;
                `;
                starfield.appendChild(star);
            }
        }

        async function init() {
            const container = document.getElementById('container');
            const containerWidth = container.clientWidth;
            const containerHeight = container.clientHeight;

            scene = new THREE.Scene();
            camera = new THREE.PerspectiveCamera(75, containerWidth / containerHeight, 0.1, 1000);
            camera.position.z = 20;

            renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true, clearColor: 0x250d63, clearAlpha: 1 });
            renderer.setSize(containerWidth, containerHeight);
            renderer.domElement.style.position = 'absolute';
            renderer.domElement.style.top = '0';
            renderer.domElement.style.left = '0';
            renderer.domElement.style.zIndex = '2';
            container.appendChild(renderer.domElement);

            // 创建星空背景
            // createStarfield();

            // 1. 解析模型获取坐标
            try {
                document.getElementById('loading').innerText = "正在提取模型表面顶点...";
                applePoints = await sampleModelSurface(APPLE_MODEL_URL, 30, APPLE_PARTICLE_COUNT); // 苹果模型
                treePoints = await sampleModelSurface(TREE_MODEL_URL, 16, TREE_PARTICLE_COUNT);   // 圣诞树模型
                christmasPoints = generateChristmasText(CHRISTMAS_PARTICLE_COUNT);                // 文字
            } catch (e) {
                console.warn("模型加载失败，切换至数学几何模式");
                applePoints = generateAppleCoords(APPLE_PARTICLE_COUNT);
                treePoints = generateTreeCoords(TREE_PARTICLE_COUNT);
                christmasPoints = generateChristmasText(CHRISTMAS_PARTICLE_COUNT);
            }

            currentParticleCount = APPLE_PARTICLE_COUNT; // 初始使用苹果粒子数

            document.getElementById('loading').style.display = 'none';

            // 2. 创建粒子
            const geometry = new THREE.BufferGeometry();
            const initialPos = new Float32Array(currentParticleCount * 3);
            // 初始状态：散开在四周
            for(let i=0; i<currentParticleCount*3; i++) initialPos[i] = (Math.random()-0.5) * 100;
            
            geometry.setAttribute('position', new THREE.BufferAttribute(initialPos, 3));

            // 初始化颜色数组
            particleColors = new Float32Array(currentParticleCount * 3);
            for(let i = 0; i < currentParticleCount; i++) {
                particleColors[i * 3] = 0.5;     // R
                particleColors[i * 3 + 1] = 0.4; // G
                particleColors[i * 3 + 2] = 0.1; // B (初始为亮金色)
            }
            geometry.setAttribute('color', new THREE.BufferAttribute(particleColors, 3));

            const material = new THREE.PointsMaterial({
                size: 0.3,                  // 增大粒子大小
                color: 0xffcc33,            // 亮金色
                transparent: true,
                opacity: 0.25,               // 降低透明度使粒子更暗
                blending: THREE.AdditiveBlending, // 叠加模式，重叠处会发白发亮
                depthWrite: false,          // 关键：防止粒子间的遮挡黑边
                map: createMixedTexture(), // 使用混合纹理（圆形和立方形）
                vertexColors: true,        // 启用顶点颜色
            });

            particles = new THREE.Points(geometry, material);
            scene.add(particles);

            // 初始汇聚动画
            transitionTo(applePoints);

            window.addEventListener('click', () => {
                // 循环切换：apple -> tree -> christmas -> apple
                if (currentShape === 'apple') {
                    currentShape = 'tree';
                } else if (currentShape === 'tree') {
                    currentShape = 'christmas';
                } else {
                    currentShape = 'apple';
                }
                
                let targetPoints;
                let targetCount;
                if (currentShape === 'apple') {
                    targetPoints = applePoints;
                    targetCount = APPLE_PARTICLE_COUNT;
                } else if (currentShape === 'tree') {
                    targetPoints = treePoints;
                    targetCount = TREE_PARTICLE_COUNT;
                } else {
                    targetPoints = christmasPoints;
                    targetCount = CHRISTMAS_PARTICLE_COUNT;
                }
                
                // 如果粒子数不同，需要重建几何体
                if (targetCount !== currentParticleCount) {
                    rebuildGeometry(targetPoints, targetCount);
                } else {
                    // 直接过渡到目标形状，不重建几何体
                    transitionTo(targetPoints);
                }
            });

            animate();
            // --- 后期处理设置 ---
            const renderScene = new THREE.RenderPass(scene, camera);
            renderScene.clearColor = new THREE.Color(0x151028);
            renderScene.clearAlpha = 1;

            // 参数：(分辨率, 强度, 半径, 阈值)
            const bloomPass = new THREE.UnrealBloomPass(
                new THREE.Vector2(window.innerWidth, window.innerHeight), 
                1.5,  // 辉光强度，降低为更柔和的效果
                0.2,  // 辉光半径，缩小发光范围
                0.2   // 辉光阈值，提高阈值让发光更集中
            );

            composer = new THREE.EffectComposer(renderer);
            composer.addPass(renderScene);
            composer.addPass(bloomPass);
        }

        // --- 核心：模型表面采样函数 ---
        function sampleModelSurface(url, scale, count) {
            return new Promise((resolve, reject) => {
                const loader = new THREE.GLTFLoader();
                loader.load(url, (gltf) => {
                    // 找到第一个有效的网格 (mesh)
                    let mesh = null;
                    gltf.scene.traverse((child) => {
                        if (child.isMesh && !mesh) {
                            mesh = child;
                        }
                    });

                    if (!mesh) {
                        reject(new Error('未找到模型中的网格'));
                        return;
                    }

                    try {
                        const sampler = new THREE.MeshSurfaceSampler(mesh).build();
                        const tempPosition = new THREE.Vector3();
                        const points = [];

                        for (let i = 0; i < count; i++) {
                            sampler.sample(tempPosition);
                            points.push(tempPosition.x * scale, tempPosition.y * scale, tempPosition.z * scale);
                        }
                        resolve(points);
                    } catch (error) {
                        reject(error);
                    }
                }, undefined, reject);
            });
        }

        function transitionTo(targetArray) {
            const posAttrib = particles.geometry.attributes.position;
            const colorAttrib = particles.geometry.attributes.color;
            const currentCount = posAttrib.array.length / 3; // 获取当前实际粒子数
            
            for (let i = 0; i < currentCount; i++) {
                gsap.to(posAttrib.array, {
                    duration: 2 + Math.random() * 2,
                    [i * 3]: targetArray[i * 3],
                    [i * 3 + 1]: targetArray[i * 3 + 1],
                    [i * 3 + 2]: targetArray[i * 3 + 2],
                    ease: "expo.inOut",
                    onUpdate: () => posAttrib.needsUpdate = true
                });
            }

            // 根据形状更新颜色
            if (currentShape === 'tree') {
                // 圣诞树：只有顶部10%格外亮，其余部分保持统一亮度
                for (let i = 0; i < currentCount; i++) {
                    const yPos = targetArray[i * 3 + 1];
                    const normalizedY = (yPos + 8) / 16; // 归一化 Y 坐标到 0-1
                    
                    let brightness;
                    if (normalizedY >= 0.9) {
                        // 顶部10%：从0.9亮度增加到1.0亮度
                        brightness = 1.0 + (normalizedY - 0.9) * 10;
                    } else {
                        // 其余部分：保持0.7的亮度
                        brightness = 0.9;
                    }
                    
                    gsap.to(colorAttrib.array, {
                        duration: 2 + Math.random() * 2,
                        [i * 3]: brightness,     // R
                        [i * 3 + 1]: brightness * 0.8, // G
                        [i * 3 + 2]: brightness * 0.2, // B
                        ease: "expo.inOut",
                        onUpdate: () => colorAttrib.needsUpdate = true
                    });
                }
            } else if (currentShape === 'christmas') {
                // "Merry Christmas" 文字：金色光点
                for (let i = 0; i < currentCount; i++) {
                    gsap.to(colorAttrib.array, {
                        duration: 2 + Math.random() * 2,
                        [i * 3]: 1.0 * 0.5,     // R
                        [i * 3 + 1]: 0.8 * 0.5, // G
                        [i * 3 + 2]: 0.2 * 0.5, // B
                        ease: "expo.inOut",
                        onUpdate: () => colorAttrib.needsUpdate = true
                    });
                }
            } else {
                // 苹果：统一亮金色，增加透明度
                for (let i = 0; i < currentCount; i++) {
                    gsap.to(colorAttrib.array, {
                        duration: 2 + Math.random() * 2,
                        [i * 3]: 1.0 * 0.5,     // R
                        [i * 3 + 1]: 0.8 * 0.5, // G
                        [i * 3 + 2]: 0.2 * 0.5, // B
                        ease: "expo.inOut",
                        onUpdate: () => colorAttrib.needsUpdate = true
                    });
                }
            }
        }

        function rebuildGeometry(targetArray, newCount) {
            // 移除旧的粒子对象
            scene.remove(particles);
            currentParticleCount = newCount; // 更新当前粒子数

            // 创建新的几何体和材质
            const geometry = new THREE.BufferGeometry();
            const initialPos = new Float32Array(newCount * 3);
            
            // 初始状态：散开在四周
            for(let i = 0; i < newCount * 3; i++) {
                initialPos[i] = (Math.random() - 0.5) * 100;
            }
            
            geometry.setAttribute('position', new THREE.BufferAttribute(initialPos, 3));

            // 初始化颜色数组
            particleColors = new Float32Array(newCount * 3);
            for(let i = 0; i < newCount; i++) {
                particleColors[i * 3] = 0.5;     // R
                particleColors[i * 3 + 1] = 0.4; // G
                particleColors[i * 3 + 2] = 0.1; // B
            }
            geometry.setAttribute('color', new THREE.BufferAttribute(particleColors, 3));

            const material = new THREE.PointsMaterial({
                size: 0.15,   // 增大粒子大小，边缘更清晰
                color: 0xffcc33,
                transparent: true,
                opacity: 0.25,  // 增加透明度
                blending: THREE.AdditiveBlending,
                depthWrite: false,
                map: createMixedTexture(),
                vertexColors: true,
            });

            particles = new THREE.Points(geometry, material);
            scene.add(particles);

            // 过渡到目标形状
            transitionTo(targetArray);
        }

        // 辅助：生成圣诞树坐标 (基于几何分布)
        function generateTreeCoords(count) {
            const pts = [];
            for (let i = 0; i < count; i++) {
                const h = Math.random() * 16 - 8;
                const r = (8 - h) * 0.4 * Math.random();
                const a = Math.random() * Math.PI * 2;
                pts.push(Math.cos(a)*r, h, Math.sin(a)*r);
            }
            return pts;
        }

        // 辅助：生成苹果坐标 (数学保底方案)
        function generateAppleCoords(count) {
            const pts = [];
            for (let i = 0; i < count; i++) {
                const phi = Math.random() * Math.PI * 2;
                const theta = Math.random() * Math.PI;
                const r = 6;
                pts.push(
                    r * Math.sin(theta) * Math.cos(phi),
                    r * Math.cos(theta) * 1.2 + Math.pow(Math.abs(Math.sin(theta)*Math.cos(phi)), 2),
                    r * Math.sin(theta) * Math.sin(phi)
                );
            }
            return pts;
        }

        // 辅助：生成"Merry Christmas"文字坐标
        function generateChristmasText(count) {
            const pts = [];
            const text = "Merry Christmas";
            const canvas = document.createElement('canvas');
            canvas.width = 1200;
            canvas.height = 300;
            const ctx = canvas.getContext('2d');
            
            // 绘制文字（减小字体）
            ctx.font = 'bold 120px Arial';
            ctx.fillStyle = 'white';
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            ctx.fillText(text, 600, 150);
            
            // 获取像素数据
            const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
            const data = imageData.data;
            const pixels = [];
            
            for (let i = 0; i < data.length; i += 4) {
                // 如果像素不透明（alpha > 128）
                if (data[i + 3] > 128) {
                    const pixelIndex = i / 4;
                    pixels.push(pixelIndex);
                }
            }
            
            // 从像素中随机采样
            for (let i = 0; i < count; i++) {
                const pixelIndex = pixels[Math.floor(Math.random() * pixels.length)];
                const pixelX = pixelIndex % canvas.width;
                const pixelY = Math.floor(pixelIndex / canvas.width);
                
                // 转换坐标到 3D 空间，并加入随机扰动（减小扰动范围）
                // 翻转 X 坐标使文字正对着我们
                const x = -(pixelX / canvas.width - 0.5) * 24 + (Math.random() - 0.5) * 0.1;
                const y = -(pixelY / canvas.height - 0.5) * 10 + (Math.random() - 0.5) * 0.1;
                const z = (Math.random() - 0.5) * 0.5;
                
                pts.push(x, y, z);
            }
            
            return pts;
        }

        function createCircleTexture() {
            const canvas = document.createElement('canvas');
            canvas.width = 64; canvas.height = 64;
            const ctx = canvas.getContext('2d');
            const grad = ctx.createRadialGradient(32,32,0, 32,32,32);
            grad.addColorStop(0, 'white');
            grad.addColorStop(1, 'transparent');
            ctx.fillStyle = grad;
            ctx.fillRect(0,0,64,64);
            return new THREE.CanvasTexture(canvas);
        }

        function createCubeTexture() {
            const canvas = document.createElement('canvas');
            canvas.width = 64; canvas.height = 64;
            const ctx = canvas.getContext('2d');
            // 绘制一个方形，中心不透明，边缘逐渐透明
            const grad = ctx.createLinearGradient(0, 0, 64, 64);
            grad.addColorStop(0, 'white');
            grad.addColorStop(0.5, 'white');
            grad.addColorStop(1, 'transparent');
            
            // 绘制方形框
            ctx.fillStyle = 'white';
            ctx.fillRect(16, 16, 32, 32); // 中心方形
            
            // 添加边缘渐变透明效果
            ctx.globalAlpha = 0.5;
            ctx.fillStyle = 'white';
            ctx.fillRect(8, 8, 48, 48);
            ctx.globalAlpha = 1.0;
            
            return new THREE.CanvasTexture(canvas);
        }

        function createMixedTexture() {
            // 随机选择圆形或立方形纹理
            return Math.random() > 0.5 ? createCircleTexture() : createCubeTexture();
        }

        function animate() {
            requestAnimationFrame(animate);
            if(particles) particles.rotation.y += 0.002;
            // renderer.clearColor(); // 清除为透明背景
            if(composer) {
                composer.render(); 
            } else {
                renderer.render(scene, camera);
            }
        }

        // 添加闪烁动画样式
        const style = document.createElement('style');
        style.innerHTML = `
            @keyframes twinkle {
                0%, 100% { opacity:0.25; }
                50% { opacity: 0.25; }
            }
        `;
        document.head.appendChild(style);
    </script>
</body>
</html>