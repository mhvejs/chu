```
import java.awt.*;
Robot robot;

int grid = 100;
PVector bpos = new PVector();
float bsize = grid;
float bspeedX, bspeedY, bspeedZ;
boolean input1, input2, input3, input4;
float cameraRotateX;
float cameraRotateY;
float cameraSpeed;
int gridCount = 50;
PVector pos, speed;
float accelMag;

void setup() {
  fullScreen(P3D);
  smooth();
  cameraSpeed = TWO_PI / width;
  cameraRotateY = -PI/6;
  pos = new PVector();
  speed = new PVector();
  accelMag = 2;
  try {
    robot = new Robot();
  } 
  catch(Exception ex) {
    println(ex);
  }
}

float angle;

void draw() {
  updateRotation();
  lights();
  translate(width/2, height/10);
  rotateX(cameraRotateY);
  rotateY(cameraRotateX);
  background(125); 
  pushMatrix();
  translate(bpos.x, height/2 + bpos.y, bpos.z);
  stroke(255);
  fill(0);
  rotateY(atan2(speed.x, speed.y));
  box(bsize);
  popMatrix();

  PVector accel = getMovementDir().rotate(cameraRotateX).mult(accelMag);
  speed.add(accel);
  pos.add(speed);
  speed.mult(0.9);  
  translate(0, height/2+bsize/2);
  drawGrid(gridCount);
  noCursor();

  angle += mouseChangeX()*0.003;
  robot.mouseMove(getSketchCenterX(), getSketchCenterY());
}

void drawGrid(int count) {
  translate(-pos.x, 0, -pos.y);
  stroke(255);
  float size = (count -1) * bsize*2;
  for (int i = 0; i < count; i++) {
    float pos = map(i, 0, count-1, -0.5 * size, 0.5 * size);
    line(pos, 0, -size/2, pos, 0, size/2);
    line(-size/2, 0, pos, size/2, 0, pos);
  }
}

void mouseClicked() //only used so I can keep track of coordinates in case I need to change things
{
  println("bpos.x" + bpos.x);
  println("bpos.y" + bpos.y);
  println("bpos.z" + bpos.z);

  println("X" + mouseX);
  println("Y" + mouseY);
}

  int getSketchCenterX() {
    return getFrame(getSurface()).getX() + width / 2;
  }
  
  int getSketchCenterY() {
    return getFrame(getSurface()).getY() + height / 2;
  }
  
  static final com.jogamp.newt.opengl.GLWindow getFrame(final PSurface surface) {
    return ((com.jogamp.newt.opengl.GLWindow) surface.getNative());
  }
  
  float mouseChangeX() {
    return getSketchCenterX() - (float) MouseInfo.getPointerInfo().getLocation().getX();
  }
  
  float mouseChangeY() {
    return getSketchCenterY() - (float) MouseInfo.getPointerInfo().getLocation().getY();
  }
  
void updateRotation() {
  cameraRotateX -= mouseChangeX() * cameraSpeed;
  cameraRotateY += mouseChangeY() * cameraSpeed;
  cameraRotateY = constrain(cameraRotateY, -HALF_PI, 0);
}

PVector getMovementDir() {
  return pressedDir.copy().normalize();
}

boolean wPressed, sPressed, aPressed, dPressed;
PVector pressedDir = new PVector();

void keyPressed() {
  switch(key) {
  case 'w':
    wPressed = true;
    pressedDir.y = -1;
    break;
  case 's':
    sPressed = true;
    pressedDir.y = 1;
    break;
  case 'a':
    aPressed = true;
    pressedDir.x = -1;
    break;
  case 'd':
    dPressed = true;
    pressedDir.x = 1;
    break;
  }
}

void keyReleased() {
  switch(key) {
  case 'w':
    wPressed = false;
    pressedDir.y = sPressed ? 1 : 0;
    break;
  case 's':
    sPressed = false;
    pressedDir.y = wPressed ? -1 : 0;
    break;
  case 'a':
    aPressed = false;
    pressedDir.x = dPressed ? 1 : 0;
    break;
  case 'd':
    dPressed = false;
    pressedDir.x = aPressed ? -1 : 0;
    break;
  }
}
