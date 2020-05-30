# 忽略电池优化的设置

1、增加权限

```
<uses-permission android:name="android.permission.REQUEST_IGNORE_BATTERY_OPTIMIZATIONS"/>
```
2、代码跳转设置，Activity中操作
```
    /**
     * 忽略电池优化
     */
    @RequiresApi(api = Build.VERSION_CODES.M)
    public void ignoreBatteryOptimization() {
        try{
            PowerManager powerManager = (PowerManager) getSystemService(POWER_SERVICE);
            boolean hasIgnored = powerManager.isIgnoringBatteryOptimizations(this.getPackageName());
            if(!hasIgnored) {
                Intent intent = new Intent(Settings.ACTION_REQUEST_IGNORE_BATTERY_OPTIMIZATIONS);
                intent.setData(Uri.parse("package:"+getPackageName()));
                startActivity(intent);
            }
        }catch(Exception e){
            //TODO :handle exception
            Log.e("ex",e.getMessage());
        }
    }
```

