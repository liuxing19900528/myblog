**马哥的淘宝店:https://shop592330910.taobao.com/**


##总结
观看马士兵老师讲课的坦克大战 视频的源代码。
学编程要亲自敲写代码，不要照抄代码，要跟着思路总结去写代码，就像你定义的类名，变量名等都可以不一样。

```
    在这几个方法里面遍历一个集合的时候，遍历的同时还要删除某个元素，这里要特别注意写法，要用迭代器。其他的方法会报错的。
    
        drawExplode(g);
        drawEnemyTanks(g);
        drawMissiles(g);
        
```
      

##坦克大战游戏的主类
```
package tank.war.client.com;

import java.awt.*;
import java.awt.event.KeyAdapter;
import java.awt.event.KeyEvent;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.Random;

public class TankClient extends Frame {
    private static final String title = "大战";

    private Tank tank = new Tank(50, 50, true, this);

    public ArrayList<Missile> missileList = new ArrayList<Missile>();
    public ArrayList<Explode> explodeList = new ArrayList<Explode>();
    public ArrayList<Tank> enemyTankList = new ArrayList<Tank>();
    public ArrayList<Wall> wallList = new ArrayList<Wall>();

    public Blood blood = new Blood();

    public static final int GAME_WIDTH = 800;
    public static final int GAME_HEIGHT = 600;

    private static final int x = 0;
    private static final int y = 0;

    private static final Color GAME_BACKGROUND = Color.lightGray;

    private Image offScreenImage = null;


    private void launchFrame() {
        createEnemyTanks(50);
        createWalls();

        this.setLocation(x, y);
        this.setSize(GAME_WIDTH, GAME_HEIGHT);
        this.setBackground(GAME_BACKGROUND);
        this.setTitle(title);
        this.setResizable(false);
        this.addWindowListener(new WindowsMonitor());
        this.addKeyListener(new KeyMonitor());
        setVisible(true);
        new Thread(new PaintThread()).start();
    }

    private void createWalls() {
        wallList.add(new Wall(100, 150, 20, 250, this));
        wallList.add(new Wall(300, 400, 300, 20, this));
    }

    private void createEnemyTanks(int count) {
        for (int i = 0; i <= count; i++) {
            Tank t = new Tank(50 + 40 * (i + 1), new Random().nextInt(GAME_HEIGHT), false, this);
            Iterator<Wall> wallIterator = wallList.iterator();
            while (wallIterator.hasNext()) {
                Wall wall = wallIterator.next();
                if (!tank.getRectangle().intersects(wall.getRectangle())) {
                    t.setGood(false);
                    t.setDirection(Tank.Direction.D);
                    this.enemyTankList.add(t);
                }
            }
        }
    }

    @Override
    public void update(Graphics g) {
        if (offScreenImage == null) {
            offScreenImage = this.createImage(GAME_WIDTH, GAME_HEIGHT);
        }
        Graphics gOffScreen = offScreenImage.getGraphics();
        Color c = gOffScreen.getColor();
        gOffScreen.setColor(GAME_BACKGROUND);
        gOffScreen.fillRect(0, 0, GAME_WIDTH, GAME_HEIGHT);
        gOffScreen.setColor(c);
        paint(gOffScreen);
        g.drawImage(offScreenImage, 0, 0, null);

    }

    @Override
    public void paint(Graphics g) {
        super.paint(g);
        drawMainTank(g);
        drawEnemyTanks(g);
        drawMissiles(g);
        drawExplode(g);
        drawString(g);
        drawWall(g);
        drawBlood(g);
    }

    private void drawMainTank(Graphics g) {
        tank.draw(g);
    }

    private void drawBlood(Graphics g) {
        if (blood.isLive()) {
            tank.eatBlood(blood);
            blood.draw(g);
        }
    }

    private void drawString(Graphics g) {
        g.drawString("Missile size: " + this.missileList.size(), 10, 50);
        g.drawString("Explode size: " + this.explodeList.size(), 10, 70);
        g.drawString("Tank    size: " + this.enemyTankList.size(), 10, 90);
        g.drawString("life        : " + this.tank.getLife(), 10, 110);
    }

    private void drawWall(Graphics g) {
        Iterator<Wall> wallIterator = wallList.iterator();
        while (wallIterator.hasNext()) {
            Wall wall = wallIterator.next();
            wall.draw(g);
        }
    }


    private void drawExplode(Graphics g) {
        Iterator<Explode> explodeIterator = explodeList.iterator();
        while (explodeIterator.hasNext()) {
            Explode explode = explodeIterator.next();
            if (!explode.isLive()) {
                explodeIterator.remove();
            } else {
                explode.draw(g);
            }
        }
    }

    private void drawMissiles(Graphics g) {
        Iterator<Missile> missileIterator = missileList.iterator();
        while (missileIterator.hasNext()) {
            Missile missile = missileIterator.next();

            if (!missile.isLive()) {
                missileIterator.remove();
            } else {
                missile.hitTanks(enemyTankList);
                missile.hitWalls(wallList);
                missile.hitTank(tank);
                missile.draw(g);
            }
        }
    }

    private void drawEnemyTanks(Graphics g) {
        if (enemyTankList.size() <= 10) {
            createEnemyTanks(20);
        }

        Iterator<Tank> tankIterator = enemyTankList.iterator();
        while (tankIterator.hasNext()) {
            Tank enemyTank = tankIterator.next();
            if (!enemyTank.isLive()) {
                tankIterator.remove();
            } else {
                enemyTank.hitWalls(wallList);
                enemyTank.hitTanks(enemyTankList);
                enemyTank.draw(g);
            }
        }
    }


    private class WindowsMonitor extends WindowAdapter {
        @Override
        public void windowClosing(WindowEvent e) {
            super.windowClosing(e);
            System.exit(0);
        }
    }

    private class PaintThread implements Runnable {

        @Override
        public void run() {
            while (true) {
                repaint();
                try {
                    Thread.sleep(50);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        }
    }

    private class KeyMonitor extends KeyAdapter {
        @Override
        public void keyPressed(KeyEvent e) {
            super.keyPressed(e);
            tank.keyPressed(e);
        }

        @Override
        public void keyReleased(KeyEvent e) {
            super.keyReleased(e);
            tank.keyReleased(e);
        }
    }

    public static void main(String[] args) {
        TankClient tankClient = new TankClient();
        tankClient.launchFrame();
    }//end main()

    public void addToMissileList(Missile missile) {
        this.missileList.add(missile);
    }

    public ArrayList<Missile> getMissileList() {
        return missileList;
    }

    public void setMissileList(ArrayList<Missile> missileList) {
        this.missileList = missileList;
    }
}

```
##坦克大战Tank类
```
package tank.war.client.com;

import java.awt.*;
import java.awt.event.KeyEvent;
import java.io.Serializable;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.Random;


public class Tank implements Serializable {
    private static final int X_SPEED = 5;
    private static final int Y_SPEED = 5;
    private static Random random = new Random();
    private static final int WIDTH = 30;
    private static final int HEIGHT = 30;
    private boolean good;
    private boolean live = true;
    private boolean bleft = false;
    private boolean bup = false;
    private boolean bright = false;
    private boolean bdown = false;

    private int step = random.nextInt(50) + 3;

    private int life = 100;

    private int x, y;
    private int oldX, oldY;

    private Direction direction = Direction.STOP;
    private Direction ptDirection = Direction.D;

    private TankClient tankClient = null;

    enum Direction {L, LU, U, RU, R, RD, D, LD, STOP}

    public Tank() {

    }

    public Tank(int x, int y, boolean good) {
        this.x = x;
        this.y = y;
        this.oldX = x;
        this.oldY = y;
        this.good = good;
    }

    public Tank(int x, int y, boolean good, TankClient tankClient) {
        this(x, y, good);
        this.tankClient = tankClient;
    }

    public Missile fire() {
        if (!live) {
            return null;
        }
        Missile m = new Missile(x, y, WIDTH, HEIGHT, ptDirection);
        m.setGood(this.good);
        m.setTankClient(tankClient);
        return m;
    }

    public ArrayList<Missile> supperFire() {
        if (!live) {
            return null;
        }
        ArrayList<Missile> list = new ArrayList<Missile>();
        Direction[] dirs = Direction.values();
        for (Direction d : dirs) {
            if (d == Direction.STOP) {
                continue;
            }
            Missile m = new Missile(x, y, WIDTH, HEIGHT, d);
            m.setGood(this.good);
            m.setColor(Color.cyan);
            m.setTankClient(tankClient);

            list.add(m);
        }
        return list;
    }

    private void move() {
        this.oldX = this.x;
        this.oldY = this.y;
        switch (direction) {
            case LU:
                indecreaseX(-X_SPEED);
                indecreaseY(-Y_SPEED);
                break;
            case RU:
                indecreaseX(X_SPEED);
                indecreaseY(-Y_SPEED);
                break;
            case LD:
                indecreaseX(-X_SPEED);
                indecreaseY(Y_SPEED);
                break;
            case RD:
                indecreaseX(X_SPEED);
                indecreaseY(Y_SPEED);
                break;
            case U:
                indecreaseY(-Y_SPEED);
                break;
            case D:
                indecreaseY(Y_SPEED);
                break;
            case L:
                indecreaseX(-X_SPEED);
                break;
            case R:
                indecreaseX(X_SPEED);
                break;
            case STOP:
                break;
        }

        if (this.direction != Direction.STOP) {
            this.ptDirection = this.direction;
        }
        if (x < 0) {
            x = 0;
            setDirection(getOppositeDirection(direction));
        }
        if (y < 0) {
            y = 0;
            setDirection(getOppositeDirection(direction));
        }
        if ((x + Tank.WIDTH) > TankClient.GAME_WIDTH) {
            x = TankClient.GAME_WIDTH - Tank.WIDTH;
            setDirection(getOppositeDirection(direction));
        }
        if ((y + Tank.HEIGHT) > TankClient.GAME_HEIGHT) {
            y = TankClient.GAME_HEIGHT - Tank.HEIGHT;
            setDirection(getOppositeDirection(direction));
        }

        if (!good) {
            if (step == 0) {
                step = random.nextInt(40) + 10;
                Direction[] dirs = Direction.values();
                int index = random.nextInt(dirs.length);
                Direction randomDirection = dirs[index];
                if (randomDirection == direction) {
                    randomDirection = getOppositeDirection(direction);
                }
                setDirection(randomDirection);

                Missile m = fire();
                if (m != null) {
                    m.setGood(false);
                    m.setColor(Color.white);
                    m.setTankClient(tankClient);
                    tankClient.addToMissileList(m);
                }
            }//end if()
            step--;

        }
    }

    public void drawPT(Graphics g) {
        g.setColor(Color.blue);
        switch (ptDirection) {
            case LU:
                g.drawLine(x + WIDTH / 2, y + HEIGHT / 2, x, y);
                break;
            case RU:
                g.drawLine(x + WIDTH / 2, y + HEIGHT / 2, x + WIDTH, y);
                break;
            case LD:
                g.drawLine(x + WIDTH / 2, y + HEIGHT / 2, x, y + HEIGHT);
                break;
            case RD:
                g.drawLine(x + WIDTH / 2, y + HEIGHT / 2, x + WIDTH, y + HEIGHT);
                break;
            case U:
                g.drawLine(x + WIDTH / 2, y + HEIGHT / 2, x + WIDTH / 2, y);
                break;
            case D:
                g.drawLine(x + WIDTH / 2, y + HEIGHT / 2, x + WIDTH / 2, y + HEIGHT);
                break;
            case L:
                g.drawLine(x + WIDTH / 2, y + HEIGHT / 2, x, y + HEIGHT / 2);
                break;
            case R:
                g.drawLine(x + WIDTH / 2, y + HEIGHT / 2, x + WIDTH, y + HEIGHT / 2);
                break;
            case STOP:
                break;
        }
    }

    public void draw(Graphics g) {
        if (!live) {
            return;
        }
        Color oldColor = g.getColor();
        if (this.good) {
            g.setColor(Color.red);
            g.fillOval(x, y, WIDTH, HEIGHT);
            g.setColor(Color.white);
            g.drawString("友", x + WIDTH / 3, y + HEIGHT / 2);

            g.setColor(Color.green);
            g.drawRect(x, y - 3, WIDTH, 3);
            int w = WIDTH * life / 100;
            g.fillRect(x, y - 3, w, 3);
        } else {
            g.setColor(Color.blue);
            g.fillOval(x, y, WIDTH, HEIGHT);
            g.setColor(Color.white);
            g.drawString("敌", x + WIDTH / 3, y + HEIGHT / 2);
        }

        drawPT(g);

        g.setColor(oldColor);


        move();
    }

    public void indecreaseY(int n) {
        this.y = this.y + n;
    }

    public void indecreaseX(int n) {
        this.x = this.x + n;
    }


    public void keyPressed(KeyEvent e) {
        int key = e.getKeyCode();
        switch (key) {

            case KeyEvent.VK_F2:
                if (!this.live && isGood()) {
                    this.live = true;
                    this.life = 100;
                }
                break;
            case KeyEvent.VK_UP:
                bup = true;
                break;
            case KeyEvent.VK_RIGHT:
                bright = true;
                break;
            case KeyEvent.VK_DOWN:
                bdown = true;
                break;
            case KeyEvent.VK_LEFT:
                bleft = true;
                break;
            default:
                break;
        }
        locateDirection();
    }

    public void keyReleased(KeyEvent e) {
        int key = e.getKeyCode();
        switch (key) {
            case KeyEvent.VK_A:
                ArrayList<Missile> list = supperFire();
                if (list != null) {
                    tankClient.getMissileList().addAll(list);
                }
                break;
            case KeyEvent.VK_CONTROL:
                Missile m = fire();
                if (m != null) {
                    tankClient.addToMissileList(m);
                }
                break;
            case KeyEvent.VK_UP:
                bup = false;
                break;
            case KeyEvent.VK_RIGHT:
                bright = false;
                break;
            case KeyEvent.VK_DOWN:
                bdown = false;
                break;
            case KeyEvent.VK_LEFT:
                bleft = false;
                break;
            default:
                break;
        }
        locateDirection();
    }

    private void locateDirection() {
        if (bleft && !bup && !bright && !bdown) {
            direction = Direction.L;
        } else if (!bleft && bup && !bright && !bdown) {
            direction = Direction.U;
        } else if (!bleft && !bup && bright && !bdown) {
            direction = Direction.R;
        } else if (!bleft && !bup && !bright && bdown) {
            direction = Direction.D;
        } else if (bleft && bup && !bright && !bdown) {
            direction = Direction.LU;
        } else if (!bleft && bup && bright && !bdown) {
            direction = Direction.RU;
        } else if (bleft && !bup && !bright && bdown) {
            direction = Direction.LD;
        } else if (!bleft && !bup && bright && bdown) {
            direction = Direction.RD;
        } else if (!bleft && !bup && !bright && !bdown) {
            direction = Direction.STOP;
        }
    }


    public boolean hitWall(Wall wall) {
        if (this.isLive()) {
            if (this.getRectangle().intersects(wall.getRectangle())) {
                this.resume();
                return true;
            }
        }
        return false;
    }

    public boolean hitWalls(ArrayList list) {
        Iterator<Wall> wallIterator = list.iterator();
        while (wallIterator.hasNext()) {
            Wall wall = wallIterator.next();
            if (hitWall(wall)) {
                return true;
            }
        }
        return false;
    }

    public boolean hitTank(Tank tank) {
        if (this.isLive() && tank.isLive() && this != tank) {
            if (this.getRectangle().intersects(tank.getRectangle())) {
                this.resume();
                tank.resume();
                return true;
            }
        }
        return false;
    }

    public boolean hitTanks(ArrayList list) {
        Iterator<Tank> tankIterator = list.iterator();
        while (tankIterator.hasNext()) {
            Tank tank = tankIterator.next();
            if (hitTank(tank)) {
                return true;
            }
        }
        return false;
    }


    public boolean eatBlood(Blood blood) {
        if (this.isLive() && blood.isLive()) {
            if (this.getRectangle().intersects(blood.getRectangle())) {
                this.setLife(100);
                blood.setLive(false);
                return true;
            }
        }

        return false;
    }


    private Direction getOppositeDirection(Direction d) {
        switch (d) {
            case LU:
                return (Direction.RD);
            case RU:
                return (Direction.LD);
            case LD:
                return (Direction.RU);
            case RD:
                return (Direction.LU);
            case U:
                return (Direction.D);
            case D:
                return (Direction.U);
            case L:
                return (Direction.R);
            case R:
                return (Direction.L);
            case STOP:
                return (Direction.L);
        }
        return d;
    }

    public Rectangle getRectangle() {
        return new Rectangle(x, y, WIDTH, HEIGHT);
    }

    public boolean isLive() {
        return live;
    }

    public void setLive(boolean live) {
        this.live = live;
    }


    public boolean isGood() {
        return good;
    }

    public void setGood(boolean good) {
        this.good = good;
    }

    public Direction getDirection() {
        return direction;
    }

    public void setDirection(Direction direction) {
        this.direction = direction;
    }

    public int getLife() {
        return life;
    }

    public void setLife(int life) {
        this.life = life;
    }

    private void resume() {
        x = oldX;
        y = oldY;
    }


}

```
##坦克大战炮弹Missile类
```
package tank.war.client.com;

import java.awt.*;
import java.util.ArrayList;
import java.util.Iterator;

public class Missile {
    private int x, y;
    private static final int X_SPEED = 15;
    private static final int Y_SPEED = 15;

    private Tank.Direction direction;

    private Color color = Color.black;
    private static final int WIDTH = 10;
    private static final int HEIGHT = 10;

    private TankClient tankClient = null;

    private boolean live = true;
    private boolean good = true;

    public Missile(int x, int y, Tank.Direction direction) {
        this.x = x;
        this.y = y;
        this.direction = direction;
    }

    public Missile(int tankX, int tankY, int tankWidth, int tankHeight,
                   Tank.Direction tankDirection) {
        this.x = tankX + tankWidth / 2 - this.WIDTH / 2;
        this.y = tankY + tankHeight / 2 - this.HEIGHT / 2;
        this.direction = tankDirection;
    }

    public void draw(Graphics g) {
        if (!live) {
            //tankClient.missileList.remove(this);
            return;
        }
        Color oldColor = g.getColor();
        g.setColor(color);
        g.fillOval(x, y, WIDTH, HEIGHT);
        g.setColor(oldColor);

        move();
    }

    private void move() {
        switch (direction) {
            case LU:
                indecreaseX(-X_SPEED);
                indecreaseY(-Y_SPEED);
                break;
            case RU:
                indecreaseX(X_SPEED);
                indecreaseY(-Y_SPEED);
                break;
            case LD:
                indecreaseX(-X_SPEED);
                indecreaseY(Y_SPEED);
                break;
            case RD:
                indecreaseX(X_SPEED);
                indecreaseY(Y_SPEED);
                break;
            case U:
                indecreaseY(-Y_SPEED);
                break;
            case D:
                indecreaseY(Y_SPEED);
                break;
            case L:
                indecreaseX(-X_SPEED);
                break;
            case R:
                indecreaseX(X_SPEED);
                break;
            case STOP:
                break;
        }
        if (x < 0 || y < 0 || x > TankClient.GAME_WIDTH || y > TankClient.GAME_HEIGHT) {
            this.setLive(false);
        }

    }

    public void indecreaseY(int n) {
        this.y = this.y + n;
    }

    public void indecreaseX(int n) {
        this.x = this.x + n;
    }

    public Rectangle getRectangle() {
        return new Rectangle(x, y, WIDTH, HEIGHT);
    }

    public boolean hitWall(Wall wall) {
        if (this.isLive()) {
            if (this.getRectangle().intersects(wall.getRectangle())) {
                this.setLive(false);
                return true;
            }
        }
        return false;
    }

    public boolean hitWalls(ArrayList list) {
        Iterator<Wall> wallIterator = list.iterator();
        while (wallIterator.hasNext()) {
            Wall wall = wallIterator.next();
            if (hitWall(wall)) {
                return true;
            }
        }
        return false;
    }

    public boolean hitTank(Tank tank) {
        if (this.isLive() && tank.isLive() && this.isGood() != tank.isGood()) {
            if (this.getRectangle().intersects(tank.getRectangle())) {
                if (tank.isGood()) {
                    tank.setLife(tank.getLife() - 20);
                    if (tank.getLife() <= 0) {
                        tank.setLive(false);
                    }
                } else {
                    tank.setLive(false);
                }

                this.setLive(false);

                tankClient.explodeList.add(new Explode(x, y, tankClient));
                return true;
            }
        }
        return false;
    }

    public boolean hitTanks(ArrayList list) {
        Iterator<Tank> tankIterator = list.iterator();
        while (tankIterator.hasNext()) {
            Tank tank = tankIterator.next();
            if (hitTank(tank)) {
                return true;
            }
        }
        return false;
    }

    public TankClient getTankClient() {
        return tankClient;
    }

    public void setTankClient(TankClient tankClient) {
        this.tankClient = tankClient;
    }

    public Color getColor() {
        return color;
    }

    public void setColor(Color color) {
        this.color = color;
    }

    public boolean isLive() {
        return live;
    }

    public void setLive(boolean live) {
        this.live = live;
    }


    public boolean isGood() {
        return good;
    }

    public void setGood(boolean good) {
        this.good = good;
    }
}//end class Missile

```
##坦克大战爆炸效果Explode 类

```
package tank.war.client.com;

import java.awt.*;

public class Explode {
    int x, y;
    private boolean live = true;

    private int[] diameter = {4, 7, 12, 18, 26, 32, 49, 30, 14, 6};
    private int step = 0;

    private TankClient tankClient;

    public Explode(int x, int y, TankClient tankClient) {
        this.x = x;
        this.y = y;
        this.tankClient = tankClient;
    }

    public void draw(Graphics g) {
        if (!live) {
            return;
        }

        if (step == diameter.length) {
            step = 0;
            live = false;
            return;
        }

        Color oldColor = g.getColor();
        g.setColor(Color.ORANGE);
        g.fillOval(x, y, diameter[step], diameter[step]);
        step++;


        g.setColor(oldColor);


    }//end draw()


    public boolean isLive() {
        return live;
    }

    public void setLive(boolean live) {
        this.live = live;
    }
}

```
##坦克大战 生命值Blood 类

```
package tank.war.client.com;

import java.awt.*;

public class Blood {
    private int x, y, width, height;
    private TankClient tankClient;
    private int[][] pos = {
            {350, 300}, {360, 300}, {375, 275}, {400, 200}, {360, 270}, {365, 290}, {340, 280}
    };
    private boolean live = true;
    private int step = 0;

    public Blood() {
        x = pos[0][0];
        y = pos[0][1];
        width = height = 15;
    }

    public void draw(Graphics g) {
        if(!live){
            return;
        }
        Color c = g.getColor();
        g.setColor(Color.PINK);
        g.fillRect(x, y, width, height);
        g.setColor(c);
        move();
    }

    private void move() {
        step++;
        if (step == pos.length) {
            step = 0;
        }
        x = pos[step][0];
        y = pos[step][1];
    }
    public Rectangle getRectangle() {
        return new Rectangle(x, y, width, height);
    }

    public boolean isLive() {
        return live;
    }

    public void setLive(boolean live) {
        this.live = live;
    }
}

```

##坦克大战 障碍物，墙Wall 类

```
package tank.war.client.com;

import java.awt.*;

public class Wall {

    private int x, y, width, height;
    private Color color;
    private TankClient tankClient = null;

    public Wall(int x, int y, int width, int height, TankClient tankClient) {
        this.x = x;
        this.y = y;
        this.width = width;
        this.height = height;
        this.tankClient = tankClient;
    }

    public void draw(Graphics g) {
        g.fillRect(x, y, width, height);
    }

    public Rectangle getRectangle() {
        return new Rectangle(x, y, width, height);
    }

}

```
