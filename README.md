```Processing
import java.awt.*;

Robot robot;

float cameraRotateX;
float cameraRotateY;
float cameraSpeed;

float angle;

boolean wPressed, sPressed, aPressed, dPressed;
PVector pressedDir = new PVector();
float accelMag;

Player player; 
Enemy enemy;

com.jogamp.newt.opengl.GLWindow myGLWindow;

float gridLevelY; 

// ----------------------------------------------------------------------------------------

void setup() {
  fullScreen(P3D);
  //size (800, 800, P3D);

  smooth(8);

  myGLWindow=getFrame(getSurface());

  accelMag = 2;
  player = new Player();
  enemy = new Enemy();

  gridLevelY = height/2; 

  cameraSpeed = TWO_PI / width;
  cameraRotateY = -PI/6;

  try {
    robot = new Robot();
  } 
  catch(Exception ex) {
    println(ex);
  }

  noCursor();
}

void draw() {
  background(125);
  lights();

  updateRotation();

  enemy.enemydisplay();
  enemy.enemymove();

  translate(width/2, height/10);
  rotateX(cameraRotateY);
  rotateY(cameraRotateX);

  player.jumpManagement(); 
  player.display(); 
  player.move();

  drawGrid();

  angle += mouseChangeX()*0.003;
  robot.mouseMove(getSketchCenterX(), getSketchCenterY());
}

// -----------------------------------------------------------------

void drawGrid() {
  int count = 50;
  translate(-player.pos.x, 
    gridLevelY, 
    -player.pos.y);
  stroke(255);
  float size = (count -1) * player.bsize*2;
  for (int i = 0; i < count; i++) {
    float pos2 = map(i, 0, count-1, -0.5 * size, 0.5 * size);
    line(pos2, 0, -size/2, pos2, 0, size/2);
    line(-size/2, 0, pos2, size/2, 0, pos2);
  }
}

// -----------------------------------------------------------------

int getSketchCenterX() {
  return
    myGLWindow.getX() + width / 2;
}

int getSketchCenterY() {
  return 
    myGLWindow.getY() + height / 2;
}

static final com.jogamp.newt.opengl.GLWindow getFrame(final PSurface surface) {
  // used only once ! 
  return 
    (com.jogamp.newt.opengl.GLWindow) surface.getNative();
}

float mouseChangeX() {
  return 
    getSketchCenterX() - (float) MouseInfo.getPointerInfo().getLocation().getX();
}

float mouseChangeY() {
  return
    getSketchCenterY() - (float) MouseInfo.getPointerInfo().getLocation().getY();
}

void updateRotation() {
  cameraRotateX -= mouseChangeX() * cameraSpeed;
  cameraRotateY += mouseChangeY() * cameraSpeed;
  cameraRotateY = constrain(cameraRotateY, -HALF_PI, 0);
}

PVector getMovementDir() {
  return 
    pressedDir.copy().normalize();
}

// --------------------------------------------------------------------------

void mouseClicked() {
  //only used so I can keep track of coordinates in case I need to change things
  player.printPos(); 
  println("X" + mouseX);
  println("Y" + mouseY);
}

void keyPressed() {
  if (keyCode == SHIFT) {
    accelMag = 4;
  }

  switch(key) {

  case ' ':
    player.startJump(); 
    break; 

  case 'w':
    wPressed = true;
    pressedDir.y = -1;
    break;

  case 'a':
    aPressed = true;
    pressedDir.x = -1;
    break;

  case 's':
    sPressed = true;
    pressedDir.y = 1;
    break;

  case 'd':
    dPressed = true;
    pressedDir.x = 1;
    break;
  }
}

void keyReleased() {
  if (keyCode == SHIFT) {
    accelMag = 2;
  }

  switch(key) {

  case 'w':
    wPressed = false;
    pressedDir.y = sPressed ? 1 : 0;
    break;

  case 'a':
    aPressed = false;
    pressedDir.x = dPressed ? 1 : 0;
    break;

  case 's':
    sPressed = false;
    pressedDir.y = wPressed ? -1 : 0;
    break;

  case 'd':
    dPressed = false;
    pressedDir.x = aPressed ? -1 : 0;
    break;
  }
}

//==================================================================

class Player {

  PVector bpos = new PVector();
  float bsize  = 100;

  // jump 
  float playermoveY = 0; 
  boolean jump=false; 
  int jumpcounter=0;

  PVector pos, speed;

  // constr 
  Player() {
    bpos.y = height/2+bsize/2;

    pos = new PVector();
    speed = new PVector();
  } // constr 

  void display() {
    pushMatrix();

    translate(bpos.x, 
      bpos.y, 
      bpos.z);
    stroke(255);
    fill(0);
    rotateY(atan2(speed.x, speed.y));
    box(bsize);

    /* // head 
     float diameter = 60; 
     fill(211, 1, 1); // RED 
     noStroke(); 
     translate(0, -diameter, 33);
     sphere(diameter/2);
     */
    popMatrix();
  }

  void move() {
    PVector accel = getMovementDir().rotate(cameraRotateX).mult(accelMag);

    speed.add(accel);
    pos.add(speed);
    speed.mult(0.9);
  }

  void startJump() {
    jumpcounter=0;
    playermoveY=-21;
    jump=true;
  }

  void jumpManagement() {
    if (jump) {
      bpos.y += 
        playermoveY;
    }

    // stop the jump
    if (bpos.y>=height/2-bsize/2) { 
      bpos.y=height/2-bsize/2;
      if (jumpcounter==0) {
        // bounce
        playermoveY=-6;
        jumpcounter++;
      } else {
        // stop jump
        jump=false;
        playermoveY=0;
      }
    } //if
    else {
      playermoveY++;
    }
  }//func 

  void printPos() {
    println("bpos.x" + bpos.x);
    println("bpos.y" + bpos.y);
    println("bpos.z" + bpos.z);
  }
  //
}
//

class Enemy {

  Enemy() {
  }
  
  void enemydisplay() {
    pushMatrix();
    fill(255,0,0);
    box(50, width/2, height/2);
    popMatrix();
  }
  
  void enemymove() {
    
  }
}
