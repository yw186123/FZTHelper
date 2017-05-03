#503快速搭建Android项目帮助工具

## 1.AbstractWelcomeActivity，欢迎界面基类，通过继承它可以实现欢迎界面的设置
- 新建一个欢迎界面layout，重写getLayoutId()方法，返回该layout；
- 重写getListener，可以在onAnimationEnd()方法中写跳转事件，在onAnimationStart()方法中做一些预处理事件；
```
@Override
    protected Animation.AnimationListener getListener() {
        return new Animation.AnimationListener() {
            @Override
            public void onAnimationStart(Animation animation) {
                Toast.makeText(AppStart.this, "动画开始啦", Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onAnimationEnd(Animation animation) {
                Toast.makeText(AppStart.this, "动画结束啦", Toast.LENGTH_SHORT).show();
                Intent intent = new Intent(AppStart.this, MainActivity.class);
                startActivity(intent);
                finish();//记得调用finish()
            }

            @Override
            public void onAnimationRepeat(Animation animation) {

            }
        };
    }
```
- 提供继承方法setDuration，可以设置动画耗时，默认为800毫秒

## 2.ExitHelper，退出帮助类，可以实现误点退出软件提示是否退出的功能
- 实例化一个ExitHelper类：
```
    private ExitHelper exitHelper;
    
    //....
    
    exitHelper = new ExitHelper(this);//初始化
    exitHelper.setBackMessage("退出咯");//设置单击退出时的提示语，也可以默认不设置
```
- 重写onKeyDown():
```
    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        return exitHelper.onKeyDown(keyCode, event);
    }
```
## 3.ConfigHelper，配置信息帮助类，单例模式，无需实例化，直接调用方法即可。可以存储一些信息到配置文件中，例如用户登录信息等等（配置文件路径在/data/data/cn.flyzy2005.fzthelper/app_config）
- 在实现Application的类中封装保存以及获取配置信息的方法：
```
       public User getUserInfo(){
        User user = new User();
        user.setId(getProperty("user_id"));
        user.setPassword(getProperty("user_password"));
        user.setUsername(getProperty("user_username"));
        return user;
    }

    public void setUserInfo(final User user){
        ConfigHelper.getInstance(this).set(new Properties(){
            {
                setProperty("user_id", user.getId());
                setProperty("user_password", user.getPassword());
                setProperty("user_username", user.getUsername());
            }
        });
    }

    private String getProperty(String key){
        return ConfigHelper.getInstance(this).get(key);
    }
```
- 通过BaseApplication在任何地方获取到配置文件中的内容：
```
    public void setUser(View view) {
        User user = new User();
        user.setId(((EditText)findViewById(R.id.user_id)).getText().toString());
        user.setPassword(((EditText)findViewById(R.id.user_password)).getText().toString());
        user.setUsername(((EditText)findViewById(R.id.user_username)).getText().toString());
        BaseApplication.getInstance().setUserInfo(user);
    }

    public void getUser(View view) {
        User user = BaseApplication.getInstance().getUserInfo();
        ((EditText)findViewById(R.id.user_id)).setText(user.getId());
        ((EditText)findViewById(R.id.user_password)).setText(user.getPassword());
        ((EditText)findViewById(R.id.user_username)).setText(user.getUsername());
    }
```