
# 8/28
깃허브에 파일 업로드 오류로 새로운 repositories 생성 후에도 오류가 나서 수정한 파일만 올려두었습니다.   
이해 안 가는 부분이나 오류 발생시 연락주세요!

## build.gradle.kts(:app)
버전 안정화 오류로 변경
```
namespace = "kr.ac.hs.farm"
    compileSdk = 35
```
에서
```
namespace = "kr.ac.hs.farm"
    compileSdk = 34
```
로 변경
```
compileOptions {
        sourceCompatibility = JavaVersion.VERSION_11
        targetCompatibility = JavaVersion.VERSION_11
    }

    kotlinOptions {
        jvmTarget = "11"
    }
```
에서
```
compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }

    kotlinOptions {
        jvmTarget = "17"
    }
```
로 변경

```
    // ML Kit 이미지 라벨링
    implementation("com.google.mlkit:image-labeling:17.0.7")
```
추가
## Tab2Activity.java
```
Log.d("러닝", "time=" + timeTextView.getText().toString());
                        Log.d("러닝", "distance=" + tvDistance.getText().toString());
                        Log.d("러닝", "kcal=" + tvKcal.getText().toString());
                        Log.d("러닝", "pace=" + tvPace.getText().toString());
```
아래에
```
// Tab3로 이번 러닝 거리 전달
                        Intent intent = new Intent(Tab2Activity.this, Tab3Activity.class);
                        double distanceKm = totalDistance; // totalDistance는 km 단위
                        intent.putExtra("lastRunDistance", distanceKm);
                        startActivity(intent);
```
추가

## Tab3Activity.java
```
import android.graphics.Bitmap;        
import android.provider.MediaStore;    
import androidx.annotation.Nullable;    
import androidx.core.app.ActivityCompat; 
```
삭제
```
import androidx.activity.result.ActivityResultLauncher;
import androidx.activity.result.contract.ActivityResultContracts;
```
추가

public class Tab3Activity extends AppCompatActivity 에

```
 private static final int REQUEST_IMAGE_CAPTURE = 1;
 private static final int REQUEST_CAMERA_PERMISSION = 100;
 private ImageView imagePreview;
 private Button buttonTakePhoto;
 private Location startLocation;
```
삭제
```
  private ProgressBar progressQuestP1,progressQuestP2;
  private ImageView boxRewardP1,boxRewardP2;
  private Button btnClaimP1,btnClaimP2;
  private ActivityResultLauncher<String> requestCameraPermissionLauncher;
  private ActivityResultLauncher<Uri> takePictureLauncher;
  private ActivityResultLauncher<Intent> previewLauncher;
  private int currentPhotoQuestNumber = -1; // P1 또는 P2
```
추가

```
imagePreview = findViewById(R.id.imagePreview);
Button buttonTakePhoto = findViewById(R.id.buttonTakePhoto);

//사진 찍기 버튼 동작 정의
        buttonTakePhoto.setOnClickListener(v -> {
            if(checkLocationDistance()){
                requestCameraPermission();
            } else{
                Toast.makeText(this,"2km 이상 이동해야 퀘스트를 수행할 수 있습니다.", Toast.LENGTH_LONG).show();
            }
        });
```
삭제

```
 //  카메라 퀘스트(P1, P2) 뷰 바인딩
        progressQuestP1 = findViewById(R.id.progressQuestP1);
        progressQuestP2 = findViewById(R.id.progressQuestP2);
        boxRewardP1 = findViewById(R.id.boxRewardP1);
        boxRewardP2 = findViewById(R.id.boxRewardP2);
        btnClaimP1 = findViewById(R.id.btnClaimP1);
        btnClaimP2 = findViewById(R.id.btnClaimP2);

        double lastRunDistance = getIntent().getDoubleExtra("lastRunDistance", 0.0);
        // Activity Result 런처 초기화
        setupActivityResultLaunchers();

        // 전달받은 러닝 거리를 1km 조건 체크에 활용
        btnClaimP1.setEnabled(lastRunDistance >= 1.0);
        btnClaimP2.setEnabled(true); // 항상 가능

        // p1번 버튼 클릭 → 권한 → 촬영 → 미리보기
        btnClaimP1.setOnClickListener(v -> {
            currentPhotoQuestNumber = 101;
            if (lastRunDistance < 1.0) {
                Toast.makeText(this, "1km 이상 러닝 시 활성화됩니다.", Toast.LENGTH_LONG).show();
                return;
            }
            ensureCameraPermissionThenCapture();
        });

        //p2번 버튼 클릭 → 권한 → 촬영 → 미리보기
        btnClaimP2.setOnClickListener(v -> {
            currentPhotoQuestNumber = 102;
            ensureCameraPermissionThenCapture();
        });

        for (int i = 0; i < claimButtons.length; i++) {
            final int index = i;
            claimButtons[i].setOnClickListener(v -> {
                claimQuest(index + 1);
            });
        }

```
추가

```
// 임시 러닝 시작 지점 설정
    private void getStartLocation() {
        startLocation = new Location("start");
        startLocation.setLatitude(37.5665);
        startLocation.setLongitude(126.9780);
    }

    //현재 위치와 시작 위치 거리 비교
    private boolean checkLocationDistance() {
        Location currentLocation = getCurrentLocation();
        if (currentLocation != null && startLocation != null) {
            float distance = startLocation.distanceTo(currentLocation);
            return distance >= 2000;
        }
        return false;
    }

    //임시 현재 위치 설정
    private Location getCurrentLocation() {
        Location location = new Location("current");
        location.setLatitude(37.5765);
        location.setLongitude(126.9880);
        return location;
    }

    //카메라 권한 요청
    private void requestCameraPermission() {
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.CAMERA}, REQUEST_CAMERA_PERMISSION);
        } else {
            openCamera();
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if (requestCode == REQUEST_CAMERA_PERMISSION) {
            if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                openCamera();
            } else {
                Toast.makeText(this, "카메라 권한이 필요합니다.", Toast.LENGTH_SHORT).show();
            }
        }
    }

    //카메라 열기 및 파일 저장 위치 지정
    private void openCamera() {
        Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
        if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
            try {
                photoFile = createImageFile();
                if (photoFile != null) {
                    photoURI = FileProvider.getUriForFile(this, getPackageName() + ".fileprovider", photoFile);
                    takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT, photoURI);
                    startActivityForResult(takePictureIntent, REQUEST_IMAGE_CAPTURE);
                }
            } catch (IOException ex) {
                ex.printStackTrace();
            }
        }
    }

    //이미지 파일 미리보기 생성
    private File createImageFile() throws IOException {
        String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss", Locale.getDefault()).format(new Date());
        String imageFileName = "JPEG_" + timeStamp + "_";
        File storageDir = getExternalFilesDir(Environment.DIRECTORY_PICTURES);
        return File.createTempFile(imageFileName, ".jpg", storageDir);
    }

    //사진 촬영 후 보상 처리 및 버튼 설정
    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);

        if (requestCode == REQUEST_IMAGE_CAPTURE && resultCode == RESULT_OK) {
            imagePreview.setImageURI(photoURI);
            buttonTakePhoto.setText("보상 받기");
            buttonTakePhoto.setOnClickListener(v -> {
                Toast.makeText(this, "보상을 받았습니다!", Toast.LENGTH_SHORT).show();
                Intent resultIntent = new Intent();
                resultIntent.putExtra("questResult", "success");
                setResult(RESULT_OK, resultIntent);
                finish();
            });
        }
    }
```
삭제

```
// ====== 카메라 퀘스트 ======

    // ★ CHANGED: Activity Result 런처 등록
    private void setupActivityResultLaunchers() {
        // 카메라 권한 요청
        requestCameraPermissionLauncher = registerForActivityResult(
                new ActivityResultContracts.RequestPermission(),
                isGranted -> {
                    if (isGranted) {
                        capturePhoto();
                    } else {
                        Toast.makeText(this, "카메라 권한이 필요합니다.", Toast.LENGTH_SHORT).show();
                    }
                }
        );

        // 사진 촬영 (URI로 저장)
        takePictureLauncher = registerForActivityResult(
                new ActivityResultContracts.TakePicture(),
                success -> {
                    if (success && photoURI != null) {
                        // 촬영 성공 → 미리보기 화면으로 이동
                        Intent intent = new Intent(this, PhotoPreviewActivity.class);
                        intent.putExtra("photoUri", photoURI);
                        intent.putExtra("questNumber", currentPhotoQuestNumber);
                        previewLauncher.launch(intent);
                    } else {
                        Toast.makeText(this, "사진 촬영이 취소되었습니다.", Toast.LENGTH_SHORT).show();
                    }
                }
        );

        // PhotoPreviewActivity 결과 수신 (보상 성공 여부)
        previewLauncher = registerForActivityResult(
                new ActivityResultContracts.StartActivityForResult(),
                result -> {
                    if (result.getResultCode() == RESULT_OK && result.getData() != null) {
                        Intent data = result.getData();
                        String status = data.getStringExtra("rewardResult");
                        int qn = data.getIntExtra("questNumber", -1);
                        if ("success".equals(status) && (qn == 101 || qn == 102)) {
                            // UI 업데이트 (상자 열림 + 진행바 100% + 버튼 비활성화)
                            handleCameraQuestRewardUI(qn);

                            // 서버에 보상 수령 요청 (기존 메서드 재사용)
                            claimQuest(qn);
                        }
                    }
                }
        );
    }

    // ★ CHANGED: 카메라 권한 확인 후 촬영 실행
    private void ensureCameraPermissionThenCapture() {
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA) == PackageManager.PERMISSION_GRANTED) {
            capturePhoto();
        } else {
            requestCameraPermissionLauncher.launch(Manifest.permission.CAMERA);
        }
    }

    // ★ CHANGED: 사진 촬영 실행 ( ACTION_IMAGE_CAPTURE + FileProvider)
    private void capturePhoto() {
        try {
            photoFile = createImageFile();
            photoURI = FileProvider.getUriForFile(this, getPackageName() + ".fileprovider", photoFile);
            takePictureLauncher.launch(photoURI);
        } catch (IOException e) {
            e.printStackTrace();
            Toast.makeText(this, "사진 파일 생성 실패", Toast.LENGTH_SHORT).show();
        }
    }

    // ★ CHANGED: 이미지 파일 생성 (기존 로직 유지)
    private File createImageFile() throws IOException {
        String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss", Locale.getDefault()).format(new Date());
        String imageFileName = "JPEG_" + timeStamp + "_";
        File storageDir = getExternalFilesDir(Environment.DIRECTORY_PICTURES);
        return File.createTempFile(imageFileName, ".jpg", storageDir);
    }

    // 보상 성공 시 UI 업데이트 (신규)
    private void handleCameraQuestRewardUI(int questNumber) {
        private void handleCameraQuestRewardUI(int questNumber) {
        try {
            int boxId;
            int progressId;
            int btnId;

            // P1(24), P2(25) 매핑
            if (questNumber == 101) {
                boxId = R.id.boxRewardP1;
                progressId = R.id.progressQuestP1;
                btnId = R.id.btnClaimP1;
            } else {
                boxId = R.id.boxRewardP2;
                progressId = R.id.progressQuestP2;
                btnId = R.id.btnClaimP2;
            }

            ImageView box = findViewById(boxId);
            if (box != null) {
                box.setImageResource(R.drawable.box_opened);
                Animation fadeIn = AnimationUtils.loadAnimation(this, R.anim.fade_open);
                box.startAnimation(fadeIn);
            }

            ProgressBar pb = findViewById(progressId);
            if (pb != null) pb.setProgress(100);

            Button btn = findViewById(btnId);
            if (btn != null) {
                btn.setEnabled(false);
                btn.setText("완료");
            }

            Toast.makeText(this, "퀘스트 " + questNumber + " 보상을 받았습니다!", Toast.LENGTH_SHORT).show();
        } catch (Exception e) {
            Log.e("CameraQuestUI", "UI update failed: " + e.getMessage());
        }
    }
    }

```
추가

## activity_tab3.xml
```
<LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal"
                android:gravity="center"
                android:layout_marginBottom="5dp"
                android:layout_marginTop="12dp">

                <ImageView
                    android:id="@+id/imagePreview"
                    android:layout_width="140dp"
                    android:layout_height="140dp"
                    android:scaleType="centerCrop"
                    android:src="@android:drawable/ic_menu_camera"
                    android:background="#E6F4EA"
                    android:contentDescription="사진 미리보기"
                    android:layout_marginTop="12dp"
                    android:layout_marginBottom="8dp"
                    android:clipToOutline="true"
                    android:outlineProvider="background"
                    android:backgroundTint="#CDE7D3" />

                <Button
                    android:id="@+id/buttonTakePhoto"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="사진 찍기"
                    android:textColor="#5D7755"
                    android:backgroundTint="#FFF7D1"
                    android:paddingLeft="24dp"
                    android:paddingRight="24dp"
                    android:paddingTop="12dp"
                    android:paddingBottom="12dp"
                    android:layout_marginStart="16dp"
                    android:elevation="4dp" />
            </LinearLayout>
```
삭제한 자리에

```
<!-- 카메라 퀘스트 영역 -->
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="10dp"
                android:layout_marginBottom="5dp"
                android:background="#BED9B8"
                android:gravity="center"
                android:orientation="vertical"
                android:paddingHorizontal="8dp">

                <!-- P1번 퀘스트 : 1km 러닝 인증 촬영 -->
                <androidx.cardview.widget.CardView
                    android:id="@+id/photoQuest1"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="10dp"
                    android:layout_marginBottom="12dp"
                    android:background="#FFFFFF"
                    app:cardCornerRadius="14dp"
                    app:cardElevation="6dp">

                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:gravity="center_vertical"
                        android:orientation="horizontal"
                        android:paddingStart="12dp"
                        android:paddingTop="16dp"
                        android:paddingEnd="12dp"
                        android:paddingBottom="16dp">

                        <ImageView
                            android:layout_width="50dp"
                            android:layout_height="50dp"
                            android:layout_marginEnd="16dp"
                            android:contentDescription="보상 이미지"
                            android:src="@android:drawable/ic_menu_camera" />

                        <!-- 텍스트 + 진행바 -->
                        <LinearLayout
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:orientation="vertical">

                            <TextView
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:fontFamily="@font/gowundodum_regular"
                                android:text="1km 러닝 인증 퀘스트"
                                android:textColor="#5D7755"
                                android:textSize="16sp"
                                android:textStyle="bold" />

                            <FrameLayout
                                android:layout_width="match_parent"
                                android:layout_height="wrap_content"
                                android:layout_marginTop="8dp">

                                <ProgressBar
                                    android:id="@+id/progressQuestP1"
                                    style="@android:style/Widget.DeviceDefault.Light.ProgressBar.Horizontal"
                                    android:layout_width="match_parent"
                                    android:layout_height="12dp"
                                    android:max="100"
                                    android:progress="0"
                                    android:progressDrawable="@drawable/progress_green_custom" />

                                <ImageView
                                    android:id="@+id/boxRewardP1"
                                    android:layout_width="24dp"
                                    android:layout_height="24dp"
                                    android:layout_gravity="end|center_vertical"
                                    android:contentDescription="보상 상자"
                                    android:src="@drawable/box_locked" />
                            </FrameLayout>
                        </LinearLayout>

                        <!-- 버튼 -->
                        <Button
                            android:id="@+id/btnClaimP1"
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            android:layout_marginStart="12dp"
                            android:backgroundTint="#FFF7D1"
                            android:elevation="2dp"
                            android:enabled="false"
                            android:text="사진찍기"
                            android:textColor="#5D7755"
                            android:textStyle="bold" />
                    </LinearLayout>
                </androidx.cardview.widget.CardView>

                <!-- P2번 퀘스트 : 나무 촬영 -->
                <androidx.cardview.widget.CardView
                    android:id="@+id/photoQuest2"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginBottom="12dp"
                    android:background="#FFFFFF"
                    app:cardCornerRadius="14dp"
                    app:cardElevation="6dp">

                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:gravity="center_vertical"
                        android:orientation="horizontal"
                        android:paddingStart="12dp"
                        android:paddingTop="16dp"
                        android:paddingEnd="12dp"
                        android:paddingBottom="16dp">

                        <ImageView
                            android:layout_width="50dp"
                            android:layout_height="50dp"
                            android:layout_marginEnd="16dp"
                            android:contentDescription="보상 이미지"
                            android:src="@android:drawable/ic_menu_camera" />

                        <!-- 텍스트 + 진행바 -->
                        <LinearLayout
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:orientation="vertical">

                            <TextView
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:fontFamily="@font/gowundodum_regular"
                                android:text="나무/식물 촬영 퀘스트"
                                android:textColor="#5D7755"
                                android:textSize="16sp"
                                android:textStyle="bold" />

                            <FrameLayout
                                android:layout_width="match_parent"
                                android:layout_height="wrap_content"
                                android:layout_marginTop="8dp">

                                <ProgressBar
                                    android:id="@+id/progressQuestP2"
                                    style="@android:style/Widget.DeviceDefault.Light.ProgressBar.Horizontal"
                                    android:layout_width="match_parent"
                                    android:layout_height="12dp"
                                    android:max="100"
                                    android:progress="0"
                                    android:progressDrawable="@drawable/progress_green_custom" />

                                <ImageView
                                    android:id="@+id/boxRewardP2"
                                    android:layout_width="24dp"
                                    android:layout_height="24dp"
                                    android:layout_gravity="end|center_vertical"
                                    android:contentDescription="보상 상자"
                                    android:src="@drawable/box_locked" />
                            </FrameLayout>
                        </LinearLayout>

                        <!-- 버튼 -->
                        <Button
                            android:id="@+id/btnClaimP2"
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            android:layout_marginStart="12dp"
                            android:backgroundTint="#FFF7D1"
                            android:elevation="2dp"
                            android:enabled="false"
                            android:text="사진찍기"
                            android:textColor="#5D7755"
                            android:textStyle="bold" />
                    </LinearLayout>
                </androidx.cardview.widget.CardView>
            </LinearLayout>
```
추가
## 신규 추가 파일 목록   
PhotoPreviewActivity.java   
activity_photopreview.xml

