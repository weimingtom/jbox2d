

# Introduction #

So getting physics right can be tough, and bugs with physics can often take a while to discover.  So it's always a good idea to create a physics prototype of the behavior you want before you build your application or game around it.  The testbed is the perfect place to to this, as it provides:
  * run loop
  * simple graphics
  * expandable settings
  * easy input support
  * serialization (saving/loading)
so you can get down to the hairy details of getting your physics correct instead of worrying about other things going wrong.  All of the current testbeds tests are [here](http://code.google.com/p/jbox2d/source/browse/#svn%2Ftrunk%2Fjbox2d-testbed%2Fsrc%2Fmain%2Fjava%2Forg%2Fjbox2d%2Ftestbed%2Ftests).

**PLEASE NOTE:** The testbed api is provided for easy prototyping and testing.  It is not intended for commercial use and might be subject to changes in the future.

# Testbed UI #

Lets start by taking a look at testbed user interface.

![http://jbox2d.googlecode.com/svn/wiki/TestbedAnnotated.png](http://jbox2d.googlecode.com/svn/wiki/TestbedAnnotated.png)

| **UI Element** | **Description** |
|:---------------|:----------------|
| Test Selection |This list contains the tests (and categories) in the testbed.  Clicking on one of these will exit the current test and enter the selected one.  Clicking on a category will start the first test in that category. |
| Engine Options | This panel contains options that modifies the way the engine behaves internally. |
| Testbed Options | These are options to modify the way the testbed or current test behaves. |
| Controls       | These buttons are pretty self-explanatory, and save/load are only enabled if the test support serialization. |
| Reporting Area |  This area is for any text from the testbed or test. |

## Mouse Controls ##

| Right click drag | moves the viewport around |
|:-----------------|:--------------------------|
| Mouse wheel      | zooms on the mouse cursor |
| Left click       | drags around physics objects in the scene (if mouse was over one) |

## Keyboard Controls ##

| **R** | restarts the current test |
|:------|:--------------------------|
| **[** | goes the last test        |
| **]** | goes to the next test     |
| **Space** | launches a 'bomb' in the test |

# Adding Tests/Settings #

Before we start writing tests, let's learn how to add them to the testbed.  The testbed has a `TestbedModel` where it stores all of the tests and settings.  Here is an example of adding our own category, test, and setting to the testbed.

```
TestbedModel model = new TestbedModel();         // create our model

// add tests
TestList.populateModel(model);                   // populate the provided testbed tests
model.addCategory("My Super Tests");             // add a category
model.addTest(new MySuperTest());                // add our test

// add our custom setting "My Range Setting", with a default value of 10, between 0 and 20
model.getSettings().addSetting(new TestbedSetting("My Range Setting", SettingType.ENGINE, 10, 0, 20));

TestbedPanel panel = new TestPanelJ2D(model);    // create our testbed panel

JFrame testbed = new TestbedFrame(model, panel); // put both into our testbed frame
// etc
testbed.setVisible(true);
testbed.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
```

Tests and settings are covered more in-depth later, but that should give you a pretty good idea how to set up your own testbed.

# A Simple Test #

Our friend 'MJW' wrote this test during the re-write to 2.1.2 to highlight a strange bug with corner collisions of polygons:

```
public class MJWTest2 extends TestbedTest {

  @Override
  public void initTest(boolean argDeserialized) {
    setTitle("Couple of Things Test");

    getWorld().setGravity(new Vec2());

    for (int i = 0; i < 2; i++) {
      PolygonShape polygonShape = new PolygonShape();
      polygonShape.setAsBox(1, 1);
      
      BodyDef bodyDef = new BodyDef();
      bodyDef.type = BodyType.DYNAMIC;
      bodyDef.position.set(5 * i, 0);
      bodyDef.angle = (float) (Math.PI / 4 * i);
      bodyDef.allowSleep = false;
      Body body = getWorld().createBody(bodyDef);
      body.createFixture(polygonShape, 5.0f);

      body.applyForce(new Vec2(-10000 * (i - 1), 0), new Vec2());
    }
  }

  @Override
  public String getTestName() {
    return "Couple of Things";
  }
}
```

The only required methods for a test are `getTestName`, which should return the test name, and 'initTest' for test initialization.  We'll go over the deserialized argument later.

# Handling Input #

The testbed provides both storage of the mouse + keyboard state and hook methods for state changes.

## Input State ##
The current input state is stored in the `TestbedModel` and in your superclass `TestbedTest`.  Here is an example of grabbing this:

```
// let's say we're in a custom test
@Override
public void step(TestbedSettings settings) {
  super.step(settings);
  
  TestbedModel model = getModel();
  if (model.getKeys()['a']) { // model also contains the coded key values
    applyLeftForce();
  }

  Vec2 screenMouse = model.getMouse();
  // ^^ this is in screen coordinates, so we'd rather grab:
  Vec2 worldMouse = super.getWorldMouse(); // which is in world coordinates

  // etc
}
```

## Hooks ##
The testbed also provides overridable hooks for input (and the world) as well.  Here are they are:
```
// engine hooks
void jointDestroyed(Joint joint);
void beginContact(Contact contact);
void endContact(Contact contact);
void postSolve(Contact contact, ContactImpulse impulse);
void preSolve(Contact contact, Manifold oldManifold);
// mouse hooks
void shiftMouseDown(Vec2 p);
void mouseUp(Vec2 p);
void mouseDown(Vec2 p);
void mouseMove(Vec2 p);
// key hooks
void keyPressed(char argKeyChar, int argKeyCode);
void keyReleased(char argKeyChar, int argKeyCode);
```

Keep in mind that the hook methods might also be used by the `TestbedTest` class, so make sure to call `super.method()` as well.

# Serialization (Saving/Loading) #
The testbed provides hooks for serialization.  With these hooks you can keep track of your references to physics entities to provide seamless serialization and deserialization.

## Enabling ##
To enable serialization, override the following method and return `true` like so:
```
@Override
public boolean isSaveLoadEnabled() {
  return true;
}
```

Keep in mind that **`initTest()` will be called every time the test is deserialized**.  If the test is deserialized, the `deserialized` argument to `initTest()` will be true, and you should not re-create any physics objects.

## Tagging ##
If your test keeps track of physics objects for anything, these references need to be updated when deserializing your test.  The testbed provides the following hooks to "tag" your object on serialization and re-assign the physics object on deserialization:

```
// for tagging
public Long getTag(Body argBody)
public Long getTag(Fixture argFixture)
public Long getTag(Joint argJoint) 
public Long getTag(Shape argShape)
public Long getTag(World argWorld)

// for re-assigning references
public void processBody(Body argBody, Long argTag)
public void processFixture(Fixture argFixture, Long argTag)
public void processJoint(Joint argJoint, Long argTag)
public void processShape(Shape argShape, Long argTag)
public void processWorld(World argWorld, Long argTag)
```

## Example ##
This is the `Vertical Stack Test`, where polygons are stacked and a 'bullet' is shot at them when the user presses the '`,`' key.  Notice the class variable `m_bullet`, and the `getTag()` and `processBody()` methods.

```
public class VerticalStack extends TestbedTest {
  private static final long BULLET_TAG = 1;
  
  public static final int e_columnCount = 5;
  public static final int e_rowCount = 16;

  Body m_bullet;
  
  @Override
  public Long getTag(Body argBody) {
    if(argBody == m_bullet){
      return BULLET_TAG;
    }
    return super.getTag(argBody);
  }
  
  @Override
  public void processBody(Body argBody, Long argTag) {
    if(argTag == BULLET_TAG){
      m_bullet = argBody;
      return;
    }
    super.processBody(argBody, argTag);
  }
  
  @Override
  public boolean isSaveLoadEnabled() {
    return true;
  }

  @Override
  public void initTest(boolean deserialized) {
    if(deserialized){
      return;
    }
    {
      BodyDef bd = new BodyDef();
      Body ground = getWorld().createBody(bd);

      PolygonShape shape = new PolygonShape();
      shape.setAsEdge(new Vec2(-40.0f, 0.0f), new Vec2(40.0f, 0.0f));
      ground.createFixture(shape, 0.0f);

      shape.setAsEdge(new Vec2(20.0f, 0.0f), new Vec2(20.0f, 20.0f));
      ground.createFixture(shape, 0.0f);
    }

    float xs[] = new float[] { 0.0f, -10.0f, -5.0f, 5.0f, 10.0f };

    for (int j = 0; j < e_columnCount; ++j) {
      PolygonShape shape = new PolygonShape();
      shape.setAsBox(0.5f, 0.5f);

      FixtureDef fd = new FixtureDef();
      fd.shape = shape;
      fd.density = 1.0f;
      fd.friction = 0.3f;

      for (int i = 0; i < e_rowCount; ++i) {
        BodyDef bd = new BodyDef();
        bd.type = BodyType.DYNAMIC;
        int n = j * e_rowCount + i;
        float x = 0.0f;
        bd.position.set(xs[j] + x, 0.752f + 1.54f * i);
        Body body = getWorld().createBody(bd);

        body.createFixture(fd);
      }
    }

    m_bullet = null;
  }

  @Override
  public void keyPressed(char argKeyChar, int argKeyCode) {
    switch (argKeyChar) {
      case ',':
        if (m_bullet != null) {
          getWorld().destroyBody(m_bullet);
          m_bullet = null;
        }

        {
          CircleShape shape = new CircleShape();
          shape.m_radius = 0.25f;

          FixtureDef fd = new FixtureDef();
          fd.shape = shape;
          fd.density = 20.0f;
          fd.restitution = 0.05f;

          BodyDef bd = new BodyDef();
          bd.type = BodyType.DYNAMIC;
          bd.bullet = true;
          bd.position.set(-31.0f, 5.0f);

          m_bullet = getWorld().createBody(bd);
          m_bullet.createFixture(fd);

          m_bullet.setLinearVelocity(new Vec2(400.0f, 0.0f));
        }
        break;
    }
  }

  @Override
  public void step(TestbedSettings settings) {
    super.step(settings);
    addTextLine("Press ',' to launch bullet.");
  }

  @Override
  public String getTestName() {
    return "Vertical Stack";
  }
}
```

# Custom Real-Time Settings #
Like mentioned [above](#Adding_Tests/Settings.md), when you initialize the testbed you can add your own real-time settings.  For example, say we would like to test adding gravity to our MJW test.  First, we want to add the setting to testbed when we create it:

```
public static final String GRAVITY_SETTING = "Gravity"; // put our setting key somewhere
```
```
TestbedModel model = new TestbedModel();         // create our model

// add tests
model.addCategory("My Tests");
model.addTest(new MJWTest2());

// add our custom setting "My Range Setting", with a default value of 10, between 0 and 20
model.getSettings().addSetting(new TestbedSetting(GRAVITY_SETTING, SettingType.ENGINE, false));

TestbedPanel panel = new TestPanelJ2D(model);    // create our testbed panel

JFrame testbed = new TestbedFrame(model, panel); // put both into our testbed frame
// etc
testbed.setVisible(true);
testbed.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
```
If we run our testbed like this, we'll see a new checkbox setting called "Gravity" in the engine settings panel. Good so far, now lets modify our `MJWTest2` a bit to use this setting...
```
public class MJWTest2 extends TestbedTest {

  @Override
  public void initTest(boolean argDeserialized) {
    setTitle("Couple of Things Test");

    getWorld().setGravity(new Vec2());

    for (int i = 0; i < 2; i++) {
      PolygonShape polygonShape = new PolygonShape();
      polygonShape.setAsBox(1, 1);
      
      BodyDef bodyDef = new BodyDef();
      bodyDef.type = BodyType.DYNAMIC;
      bodyDef.position.set(5 * i, 0);
      bodyDef.angle = (float) (Math.PI / 4 * i);
      bodyDef.allowSleep = false;
      Body body = getWorld().createBody(bodyDef);
      body.createFixture(polygonShape, 5.0f);

      body.applyForce(new Vec2(-10000 * (i - 1), 0), new Vec2());
    }
  }

  @Override
  public void step(TestbedSettings settings) {
    super.step(settings); // make sure we update the engine!
    TestbedSetting gravity = settings.getSetting(GRAVITY_SETTING); // grab our setting
    if (gravity.enabled) {
      getWorld().setGravity(new Vec2(0, -9);
    else {
      getWorld().setGravity(new Vec2());
    }
  }

  @Override
  public String getTestName() {
    return "Couple of Things";
  }
}
```

And that's all there is to it!  You just add the settings check in the `step()` function, and you're set!


---

Remember show your appreciation and [donate](http://jbox2d.org)!