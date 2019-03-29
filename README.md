# AndroidForceUpdateusingFRC (Firebase Remote Config)

1. Configure the firebase into the android application.

    * Register your app in firebase console.
    * Add the required dependencies.
    * Download the jon file and copy into application folder.
    * Sync the application and verify.
    
2. Add remote config dependencies.

```
implementation 'com.google.firebase:firebase-config:16.4.0'
```

3. Add the required parameters in remote config and publish.

   ![Image_01](/Assert/Images/Image_01.png)
   
4. AppUpadteDialog.java

```java
   
public class AppUpdateDialog extends Dialog
{
    private TextView tvTitle,tvDescription,tvUpdate;
    ImageView ivClose;
    private Context context;
    String title;
    String description;
    private boolean isCancelable;
    public AppUpdateDialog(Context context, String title, String description, boolean isCancelable)
    {
        super(context);
        // TODO Day selector
        this.context = context;
        this.title = title;
        this.description = description;
        this.isCancelable = isCancelable;
    }
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        setContentView(R.layout.view_app_update);
        tvTitle=findViewById(R.id.tvTitle);
        tvDescription=findViewById(R.id.tvMessage);
        tvUpdate=findViewById(R.id.tvUpdateNow);

        ivClose=findViewById(R.id.ivClose);

        if (isCancelable)
            ivClose.setVisibility(View.VISIBLE);
        else
            ivClose.setVisibility(View.GONE);

        ivClose.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                dismiss();
            }
        });


        if(!TextUtils.isEmpty(title))
            tvTitle.setText(title);
        else
            tvTitle.setVisibility(View.GONE);


        if(!TextUtils.isEmpty(description))
            tvDescription.setText(String.format(description));
        else
            tvDescription.setVisibility(View.GONE);

        tvUpdate.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {

                // getPackageName() from Context or Activity object
                final String appPackageName = context.getPackageName();

                try {
                    context.startActivity(new Intent(Intent.ACTION_VIEW, Uri.parse("market://details?id=" + appPackageName)));
                } catch (ActivityNotFoundException activityNotFoundException) {
                    context.startActivity(new Intent(Intent.ACTION_VIEW, Uri.parse("https://play.google.com/store/apps/details?id=" + appPackageName)));
                }
            }
        });

    }
}

```

5. MainActivity.java

```java
public class MainActivity extends AppCompatActivity
{
    private static final String FB_RC_KEY_TITLE="update_title";
    private static final String FB_RC_KEY_DESCRIPTION="update_description";
    private static final String FB_RC_KEY_FORCE_UPDATE_VERSION="force_update_version";
    private static final String FB_RC_KEY_LATEST_VERSION="latest_version";

    AppUpdateDialog appUpdateDialog;

    FirebaseRemoteConfig mFirebaseRemoteConfig;

    Timer timer;
    TimerTask timerTask;
    
       @Override
    protected void onCreate(Bundle savedInstanceState) 
    {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
       
        checkAppUpdate();
    }

 public void checkAppUpdate() {

        final int versionCode = BuildConfig.VERSION_CODE;

        final HashMap<String, Object> defaultMap = new HashMap<>();
        defaultMap.put(FB_RC_KEY_TITLE, "Update Available");
        defaultMap.put(FB_RC_KEY_DESCRIPTION, "A new version of the application is available please click below to update the latest version.");
        defaultMap.put(FB_RC_KEY_FORCE_UPDATE_VERSION, ""+versionCode);
        defaultMap.put(FB_RC_KEY_LATEST_VERSION, ""+versionCode);

        mFirebaseRemoteConfig = FirebaseRemoteConfig.getInstance();

        mFirebaseRemoteConfig.setConfigSettings(new FirebaseRemoteConfigSettings.Builder().setDeveloperModeEnabled(BuildConfig.DEBUG).build());

        mFirebaseRemoteConfig.setDefaults(defaultMap);

        Task<Void> fetchTask=mFirebaseRemoteConfig.fetch(BuildConfig.DEBUG?0: TimeUnit.HOURS.toSeconds(4));

        fetchTask.addOnCompleteListener(new OnCompleteListener<Void>() {
            @Override
            public void onComplete(@NonNull Task<Void> task) {
                if (task.isSuccessful()) {
                    // After config data is successfully fetched, it must be activated before newly fetched
                    // values are returned.
                    mFirebaseRemoteConfig.activateFetched();

                    String title=getValue(FB_RC_KEY_TITLE,defaultMap);
                    String description=getValue(FB_RC_KEY_DESCRIPTION,defaultMap);
                    int forceUpdateVersion= Integer.parseInt(getValue(FB_RC_KEY_FORCE_UPDATE_VERSION,defaultMap));
                    int latestAppVersion= Integer.parseInt(getValue(FB_RC_KEY_LATEST_VERSION,defaultMap));

                    boolean isCancelable=true;

                    if(latestAppVersion>versionCode)
                    {
                        if(forceUpdateVersion>versionCode)
                            isCancelable=true;

                        if(appVar.appUpdateShown == false) {

                            appVar.appUpdateShown = true;

                            appUpdateDialog = new AppUpdateDialog(MainActivity.this,title,description,isCancelable);
                            appUpdateDialog.setCancelable(true);
                            appUpdateDialog.show();

                            Window window = appUpdateDialog.getWindow();
                            assert window != null;
                            window.setLayout(ConstraintLayout.LayoutParams.MATCH_PARENT, ConstraintLayout.LayoutParams.WRAP_CONTENT);
                        }


                    }

                } else {
                   // Toast.makeText(MainActivity.this, "Fetch Failed",
                    //        Toast.LENGTH_SHORT).show();
                }
            }
        });
    }

    public String getValue(String parameterKey,HashMap<String, Object> defaultMap)
    {
        String value=mFirebaseRemoteConfig.getString(parameterKey);
        if(TextUtils.isEmpty(value))
            value= (String) defaultMap.get(parameterKey);

        return value;
    }
    
}
```

6. view_app_update.xml

```

<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ImageView
        android:id="@+id/ivClose"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:padding="12dp"
        android:src="@drawable/ic_close"
        app:layout_constraintRight_toRightOf="parent" />

    <ImageView
        android:id="@+id/ivApplication"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_marginTop="16dp"
        android:padding="8dp"
        android:src="@drawable/push"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />


    <TextView
        android:id="@+id/tvTitle"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="16dp"
        android:layout_marginRight="16dp"
        android:layout_marginTop="12dp"
        android:fontFamily="@font/montserrat_regular"
        android:gravity="center"
        android:text="Test"
        android:textSize="20sp"
        app:layout_constraintTop_toBottomOf="@+id/ivApplication" />

    <TextView
        android:id="@+id/tvMessage"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="20dp"
        android:layout_marginRight="20dp"
        android:layout_marginTop="32dp"
        android:fontFamily="@font/montserrat_light"
        android:gravity="center"
        android:text="@string/app_update_default_description"
        android:textSize="@dimen/xlarge_text"
        app:layout_constraintTop_toBottomOf="@+id/tvTitle" />

    <TextView
        android:id="@+id/tvUpdateNow"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="32dp"
        android:background="#7CC68D"
        android:fontFamily="@font/montserrat_light"
        android:gravity="center"
        android:padding="12dp"
        android:text="@string/app_update_positive_text"
        android:textColor="@android:color/white"
        android:textSize="@dimen/xxlarge_text"
        app:layout_constraintTop_toBottomOf="@+id/tvMessage" />

</android.support.constraint.ConstraintLayout>

```



