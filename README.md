# R4
import ddf.minim.*; 

PImage houseImg;
PImage batakoImg;
PImage waterImg;
PShape anpanmanModel;
PShape anpanmanModel2;
PShape anpanmanBodyModel;
PShape newObjectShape;
PFont font;

float imgX, imgY;
float batakoX, batakoY;
float waterX, waterY;
float waterSpeed = 20; 
float waterMaxY; 
float waterMinY; 

boolean isShooting = false;
float shootSpeed = 0;
boolean moveLeft = false;
boolean moveRight = false;

boolean isStarted = false;
int startTime; 
int timeLimit = 60000; // タイマーの制限時間（ミリ秒）

boolean waterExists = true; 
boolean isReplaced = false; 

float fallSpeedX = 0;
float fallSpeedY = 0; 
int points = 0; 

// アンパンマンの飛行に関する変数
float anpanmanX;
float anpanmanY;
float anpanmanSpeedX = 5;
float anpanmanSpeedY = 2;
float targetAngle; 
float currentAngle; 

int elapsedTime = 0; 
float anpanmanRotation = 0;


float rotationSpeedX = 0;
float rotationSpeedY = 0;
float rotationSpeedZ = 0;

// anpanmanModel2 の飛行に関する変数
boolean isFlyingUp = false;
boolean isRotating = false;
boolean hasRotated = false;
float anpanmanModel2Y;
float anpanmanModel2X;
float anpanmanModel2Rotation = 0; 

Minim minim;
AudioPlayer bgm; 
AudioPlayer shootingSound; 
AudioPlayer start;
AudioPlayer finish;
AudioPlayer wet;

void settings() {
    size(1000, 1400, P3D);
}

void setup() {
    minim = new Minim(this);
    bgm = minim.loadFile("sound_bgm.mp3"); 
    bgm.setGain(-20); 
    shootingSound = minim.loadFile("sound_shooting.mp3"); 
    start = minim.loadFile("sound_start.mp3");
    finish = minim.loadFile("sound_finish.mp3");
    wet = minim.loadFile("sound_wet.mp3");
    
    // 画像を読み込む
    houseImg = loadImage("House4.jpg");
    batakoImg = loadImage("batako2.png");
    waterImg = loadImage("water.png");
    anpanmanModel = loadShape("anpanman.obj");
    anpanmanModel2 = loadShape("anpanman.obj");
    anpanmanBodyModel = loadShape("anpanman_body.obj");
    newObjectShape = loadShape("anpanman_face_wet.obj");
    imgX = width / 2;
    imgY = height / 1.3;
    batakoX = imgX;
    batakoY = height / 1.4 + 100;
    waterX = 0; 
    waterMinY = height * 0.2; 
    waterMaxY = height * 0.8; 
    waterY = random(waterMinY, waterMaxY); 

    anpanmanX = width / 2 + random(-100, 100); 
    anpanmanY = height / 4 + random(-400, 400);; 
    anpanmanModel2X = anpanmanX + 20; 
    anpanmanModel2Y = anpanmanY; 

    font = createFont("NotoSansJP-Regular.otf", 32); 
    textFont(font);
}

void draw() {
    background(255);

    if (houseImg != null) {
        float aspectRatio = (float) houseImg.width / houseImg.height;
        float newWidth = height * aspectRatio;
        image(houseImg, (width - newWidth) / 2, 0, newWidth, height);
    } else {
        println("Failed to load House4.jpg");
    }

    if (isStarted) {
        elapsedTime = millis() - startTime;
        int remainingTime = max(0, timeLimit - elapsedTime);

        if (remainingTime > 0) {
            if (isReplaced) {
                pushMatrix();
                translate(imgX + 80, imgY + 140);
                rotateY(-HALF_PI);
                rotateX(PI);
                scale(0.5);
                shape(newObjectShape);
                popMatrix();
            } else {
                if (anpanmanModel != null) {
                    pushMatrix();
                    translate(imgX + 80, imgY + 140);
                    rotateY(-HALF_PI);
                    rotateX(PI);
                    if (isShooting) {
                        rotateX(rotationSpeedX * anpanmanRotation);
                        rotateY(rotationSpeedY * anpanmanRotation);
                        rotateZ(rotationSpeedZ * anpanmanRotation);
                        anpanmanRotation += 0.2; 
                    }
                    scale(0.5);
                    shape(anpanmanModel);
                    popMatrix();
                } else {
                    println("Failed to load anpanman.obj");
                }
            }

            // anpanmanBodyModelが空を飛ぶように設定
            if (anpanmanBodyModel != null) {
                pushMatrix();
                translate(anpanmanX, anpanmanY);
                float angle = atan2(anpanmanSpeedY, anpanmanSpeedX);
                float angle2 = atan2(anpanmanSpeedY, -anpanmanSpeedX);
                targetAngle = angle; 
                currentAngle = lerp(currentAngle, targetAngle, 0.1); 
                rotateZ(currentAngle);
                rotateX(angle2);
                scale(0.5);
                shape(anpanmanBodyModel);
                
                
                pushMatrix();
                translate(20, 0, 0); 
                if (isRotating) {
                    anpanmanModel2Rotation += 1; 
                    if (anpanmanModel2Rotation >= TWO_PI * 5) { // 5回転したら停止
                        anpanmanModel2Rotation = 0;
                        isRotating = false;
                        hasRotated = true; 
                    }
                }
                rotateY(anpanmanModel2Rotation); 
                shape(anpanmanModel2);
                popMatrix();

                popMatrix();

               
                anpanmanX += anpanmanSpeedX;
                anpanmanY += anpanmanSpeedY;

                
                if (anpanmanX < 0 || anpanmanX > width) {
                    anpanmanSpeedX *= -1;
                    anpanmanX = constrain(anpanmanX, 0, width); 
                }
                if (anpanmanY < 0 || anpanmanY > height / 2) {
                    anpanmanSpeedY *= -1;
                    anpanmanY = constrain(anpanmanY, 0, height / 2); 
                }

               
                anpanmanRotation += 0.05;
            } else {
                println("Failed to load anpanman_body.obj");
            }

            if (batakoImg != null) {
                float scaleFactor = 0.5;
                image(batakoImg, batakoX, batakoY, batakoImg.width * scaleFactor, batakoImg.height * scaleFactor);
            } else {
                println("Failed to load batako2.png");
            }

            if (waterExists && waterImg != null) {
                float fixedWidth = 100;
                float fixedHeight = 100;
                image(waterImg, waterX, waterY, fixedWidth, fixedHeight); 
               
                waterX += waterSpeed;                
                if (waterX > width) {
                    waterX = -fixedWidth; // 初期位置に戻す
                    waterY = random(waterMinY, waterMaxY - fixedHeight); // 新しいランダムなY座標を設定
                }

                // 水との衝突検出
                if (isShooting && imgX + 80 >= waterX && imgX + 80 <= waterX + fixedWidth && imgY + 140 >= waterY && imgY + 140 <= waterY + fixedHeight) {
                    waterExists = false; 
                    isReplaced = true; 
                    fallSpeedX = random(-5, 5);
                    fallSpeedY = 10; 
                    wet.rewind();
                    wet.play();
                }
            }

            if (isShooting) {
                imgY += fallSpeedY;
                imgX += fallSpeedX;

                // アンパンマンの手の位置
                float handX = anpanmanX + 50; 
                float handY = anpanmanY + 20; 

                // 手の位置との衝突検出
                float dx = imgX + 140 - handX; 
                float dy = imgY + 200 - handY; 
                float distance = sqrt(dx * dx + dy * dy); 

                if (distance < 50) { // 閾値は調整可能
                    points += 1; // ポイントを増加させる
                    isShooting = false;
                    imgX = width / 2;
                    imgY = height / 1.3;
                    batakoX = imgX;
                    batakoY = height / 1.4 + 100;
                    shootSpeed = 0;                    
                  
                    isFlyingUp = false;
                    isRotating = true;
                    hasRotated = false;
                    
                }
                if (imgY > height || imgX < 0 || imgX > width || (fallSpeedY < 0 && imgY + 140 < 0)) {
                    isShooting = false;
                    imgX = width / 2;
                    imgY = height / 1.3;
                    batakoX = imgX;
                    batakoY = height / 1.4 + 100;
                    shootSpeed = 0;
                    anpanmanRotation = 0; 
                    waterExists = true; // 水を再生成
                    waterX = 0;
                    waterY = random(waterMinY, waterMaxY);
                    isReplaced = false;
                }
            }
            if (!isShooting) {
                if (moveLeft) {
                    imgX -= 10;
                    batakoX -= 10;
                } else if (moveRight) {
                    imgX += 10;
                    batakoX += 10;
                }
            }

            // タイマーを表示
            int seconds = remainingTime / 1000;
            fill(0);
            textAlign(RIGHT, TOP);
            text("時間: " + nf(seconds, 2), width - 10, 10);

            // ポイントを表示
            textAlign(LEFT, TOP);
            text("ポイント: " + points, 10, 10);
        } else {
            isStarted = false;
            bgm.close(); // ゲーム終了時に音楽を停止
            finish.rewind();
            finish.play();
            
        }
    } 

    if (!isStarted) {
        // ゲームのタイトルを表示
        textSize(120);
        textAlign(CENTER, CENTER);
        fill(255, 0, 0);
        text("アンパンマン", width / 2, height / 2 - 300);
        textSize(60);
        fill(255, 255, 0);
        text("かおかえっこ!", width / 2, height / 2 - 200);
        textSize(32); 

        // スタートボタンを表示
        fill(255, 0, 0);
        rect(width / 2 - 100, height / 2 + 100, 200, 100);
        fill(0);
        textAlign(CENTER, CENTER);
        text("はじめる", width / 2, height / 2 + 150);

        // 終わるボタンを表示
        fill(255, 255, 0);
        rect(width / 2 - 100, height / 2 + 250, 200, 100);
        fill(0);
        textAlign(CENTER, CENTER);
        text("おわる", width / 2, height / 2 + 300);

        // 最終スコアを表示
        if (elapsedTime > 0) {
            fill(0);
            textSize(48);
            text("とくてん: " + points, width / 2, height / 2 - 100);
            textSize(32);
            
            // 点数に応じたメッセージ
            if (points >= 15) {
                text("すばらしい!!!", width / 2, height / 2 );
            } else if (points >= 10) {
                text("やったね!!", width / 2, height / 2 );
            } else {
                text("もういちどやってみよう!", width / 2, height / 2 );
            }
        }
    }
}

void keyPressed() {
    if (isStarted) {
        if (keyCode == LEFT) {
            moveLeft = true;
        } else if (keyCode == RIGHT) {
            moveRight = true;
        } else if (key == ' ') {
            if (!isShooting) { 
                isShooting = true;
                shootSpeed = 7;
                fallSpeedX = 0;
                fallSpeedY = -15;
                anpanmanRotation = 0; 
                rotationSpeedY = 3;
                shootingSound.rewind();
                shootingSound.play(); 
            }
        }
    }
}

void keyReleased() {
    if (keyCode == LEFT) {
        moveLeft = false;
    } else if (keyCode == RIGHT) {
        moveRight = false;
    }
}

void mousePressed() {
    if (!isStarted) {
        if (mouseX > width / 2 - 100 && mouseX < width / 2 + 100 && mouseY > height / 2 + 100 && mouseY < height / 2 + 200) {
            isStarted = true;
            startTime = millis();
            points = 0; 
            elapsedTime = 0; 
            bgm.loop(); 
            start.rewind();
            start.play();
        }
        
        // 終わるボタンのクリックを検出
        if (mouseX > width / 2 - 100 && mouseX < width / 2 + 100 && mouseY > height / 2 + 250 && mouseY < height / 2 + 350) {
            exit(); // プログラムを終了する
        }
    }
}
