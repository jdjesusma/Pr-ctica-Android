# Práctica-Android
# Prerequisitos 
- Descargar e instalar Android Studio desde https://developer.android.com/studio
- Crear un nuevo proyecto en Android Studio, al crearlo seleccionar la opción "No Activity"
- En la configuración del proyecto, asignar el nombre y package name deseados, para el lenguaje seleccionar Java y como SDK Mínimo seleccionar API 21 o superior: <br>
  <img width="540" alt="Captura de pantalla 2024-11-28 a la(s) 3 40 54 p m" src="https://github.com/user-attachments/assets/de163c57-a2a2-4475-ba5f-8e79b52f7d93">
- Al crear el proyecto, Android Studio comenzará a configurar y descargar las cosas necesarias, la primera vez puede tardar unos minutos.
- Crear un dispositivo vitual siguiendo los siguientes pasos:
  - Seleccionar Menú Tools -> Device Manager -> ícono "+" -> Create Virtual Device
  - Seleccionar Categoría y modelo de dispositivo
  - Seleccionar versión de Android para el dispositivo virtual, esta versión de descargará y puede tardar varios minutos.
  - Validar configuración del dispositivo a crear y seleccionar Finish.
- Crear una cuenta gratuita en mockAPI https://mockapi.io, aquí se guardarán los datos del CRUD.
- Iniciar sesión en mockAPI y crear un proyecto nuevo, se puede usar cualquier nombre y cualquier prefijo de API<br>
  <img width="320" alt="Captura de pantalla 2024-11-28 a la(s) 3 29 55 p m" src="https://github.com/user-attachments/assets/ad967796-0b18-45ef-a130-77d4947fd418">
- Seleccionar el proyecto creado y ahora crear un recurso, para esta práctica se crea el recurso `products` el cual tiene los campos `name`, `price`; ademas del id <br>
  <img width="420" alt="Captura de pantalla 2024-11-28 a la(s) 3 32 16 p m" src="https://github.com/user-attachments/assets/4c043f9c-0472-49cb-ae16-53ccef67e1d0">
- En este punto, mockApi debe mostra algo como esto: <br>
  <img width="500" alt="Captura de pantalla 2024-11-28 a la(s) 3 34 16 p m" src="https://github.com/user-attachments/assets/4675cbfa-c078-451c-ba32-039b136a35d7">
- El dato API Endpoint se usará en el código más adelante.

# Procedimiento de la práctica
- Abrir el proyecto creado en los prerequisitos, o en su caso crear un proyecto nuevo.
- Abrir el archivo `libs.version.toml` y agregar las siguientes líneas:
  ```toml libs.version.toml
  [versions]
  #...
  retrofit = "2.11.0"
  gson = "2.10.1"
  converterGson = "2.3.0"
- En el mismo archivo agregar las siguientes líneas:
  ```toml libs.version.toml
  [libraries]
  #...
  retrofit = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit" }
  gson = { group = "com.google.code.gson", name = "gson", version.ref = "gson" }
  converter-gson = { group = "com.squareup.retrofit2", name = "converter-gson", version.ref = "converterGson" }
- Las lineas mencionadas hacen referencia a las librerias externas que se usarán en el código.
- Lo siguiente es abrir el archivo `build.gradle (Module :app)` y agregar las siguientes líneas casi hasta el final del archivo:
  ``` gradle
  dependencies {
    //...
    implementation(libs.retrofit)
    implementation(libs.gson)
    implementation(libs.converter.gson)
  }
- En el mismo archivo `build.gradle (Module :app)` agregar las siguientes líneas dentro:
  ``` gradle
  android {
    //...
    buildFeatures {
        viewBinding = true
    }
  }
- En seguida, crear un nuevo paquete con el nombre `model` y dentro crear una clase con el nombre `Product`, esta clase tendrá el siguinte código:
  ```java Product.java
  public class Product implements Serializable {
      private String id;
      private String name;
      private String price;
  
      public String getId() {
          return id;
      }
  
      public void setId(String id) {
          this.id = id;
      }
  
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  
      public String getPrice() {
          return price;
      }
  
      public void setPrice(String price) {
          this.price = price;
      }
  }
- Ahora, crear un nuevo paquete con el nombre `provider` y crear dentro una nueva clase con el nombre `RetrofitProvider`, la cual tendrá el siguiente código:
  ``` Java
  public class RetrofitProvider {

      private static Retrofit retrofit;
  
      public static synchronized Retrofit getRetrofit() {
          if (retrofit == null) {
              OkHttpClient client = new OkHttpClient.Builder().build();
              retrofit = new Retrofit.Builder()
                      //TODO: Poner el API Endpoint que se generó en mockAPI, ejemplo: https://6747947f38c8741641d71c48.mockapi.io/api/v1/
                      .baseUrl("")
                      .addConverterFactory(GsonConverterFactory.create())
                      .client(client)
                      .build();
  
          }
          return retrofit;
      }
  }
- Para las conexiones a la API vamos a crear un nuevo paquete `service` y dentro creamos una interface con el nombre `ProductService`, de la cual el código se muestra en seguida:
  ```Java ProductService.java
  import java.util.List;

  import retrofit2.Call;
  import retrofit2.http.Body;
  import retrofit2.http.DELETE;
  import retrofit2.http.GET;
  import retrofit2.http.POST;
  import retrofit2.http.PUT;
  import retrofit2.http.Path;
  
  public interface ProductService {
  
      @GET("products")
      Call<List<Product>> getProductList();
  
      @GET("products/{id}")
      Call<Product> getProduct(@Path("id") String id);
  
      @POST("products")
      Call<Product> postProduct(@Body Product product);
  
      @PUT("products/{id}")
      Call<Product> putProduct(@Path("id") String id, @Body Product product);
  
      @DELETE("products/{id}")
      Call<String> deleteProduct(@Path("id") String id);
  } 
- La app tendrá una arquitectura, Model-View-Presenter, para definir la funcionalidad de cada pantalla se crea un paquete nuevo llamado `contract` y dentro creamos las tres interfaces `ProductListContract`, `ProductDetailContract` y `ProductFormContract`, los codigos de dichas interfaces se muestren en seguida:
  - ProductListContract.java
    ```Java ProductListContract.java
    public interface ProductListContract {
        interface View {
            void showProducts(List<Product> products);
        }
    
        interface Presenter {
            void getProductList();
        }
    }
  - ProductDetailContract.java
    ```Java ProductDetailContract.java
    public interface ProductDetailContract {
        public interface View {
            void onProductDeleted();
        }
    
        interface Presenter {
            void deleteProduct(String productId);
        }
    }
  - ProductFormContract.java
    ```Java ProductFormCotract.java
    public interface ProductFormContract {
        interface View {
            void onProductUpdated();
            void onProductCreated();
        }
    
        interface Presenter {
            void updateProduct(String id, Product product);
            void createProduct(Product product);
        }
    }
- En seguida se contruye la lógica de los Presenters, que son los que ejecutarán los llamados a la API, para ello se crea un nuevo paquete `presenter` y dentro creamos las clases `ProductListPresenter`, `ProductDetailPresenter` y `ProductFormPresenter`, a continuación el código de cada clase:
  - ProductListPresenter.java
    ```Java ProductListPresenter.java
    public class ProductListPresenter implements ProductListContract.Presenter {
        ProductListContract.View view;
        ProductService productService;
    
        public ProductListPresenter(ProductListContract.View view, ProductService productService) {
            this.view = view;
            this.productService = productService;
        }
    
        @Override
        public void getProductList() {
            Call<List<Product>> call = productService.getProductList();
            call.enqueue(new Callback<>() {
                @Override
                public void onResponse(Call<List<Product>> call, Response<List<Product>> response) {
                    view.showProducts(response.body());
                }
    
                @Override
                public void onFailure(Call<List<Product>> call, Throwable throwable) {
    
                }
            });
        }
    }
  - ProductDetailPresenter.java
    ```Java ProductDetailPresenter.java
    public class ProductDetailPresenter implements ProductDetailContract.Presenter {
    
        ProductDetailContract.View view;
        ProductService client;
    
        public ProductDetailPresenter(ProductDetailContract.View view, ProductService client) {
            this.view = view;
            this.client = client;
        }
    
        @Override
        public void deleteProduct(String id) {
            Call<String> call = client.deleteProduct(id);
            call.enqueue(new Callback<String>() {
                @Override
                public void onResponse(Call<String> call, Response<String> response) {
                    view.onProductDeleted();
                }
    
                @Override
                public void onFailure(Call<String> call, Throwable throwable) {
                    view.onProductDeleted();
                }
            });
        }
    }
  - ProductFormPresenter.java
    ```Java ProductFormPresenter.java
    public class ProductFormPresenter implements ProductFormContract.Presenter {
        ProductFormContract.View view;
        ProductService client;
    
        public ProductFormPresenter(ProductFormContract.View view, ProductService client) {
            this.view = view;
            this.client = client;
        }
    
        @Override
        public void updateProduct(String id, Product product) {
            Call<Product> call = client.putProduct(id, product);
            call.enqueue(new Callback<>() {
                @Override
                public void onResponse(Call<Product> call, Response<Product> response) {
                    view.onProductUpdated();
                }
    
                @Override
                public void onFailure(Call<Product> call, Throwable throwable) {
    
                }
            });
        }
    
        @Override
        public void createProduct(Product product) {
            Call<Product> call = client.postProduct(product);
            call.enqueue(new Callback<>() {
                @Override
                public void onResponse(Call<Product> call, Response<Product> response) {
                    view.onProductCreated();
                }
    
                @Override
                public void onFailure(Call<Product> call, Throwable throwable) {
    
                }
            });
        }
    }
- En la carpeta `res -> drawable`, crear un archivo nuevo con el nombre `ic_add.xml´, el cual debe tener lo siguiente:
  ```xml ic_add.xml
  <vector xmlns:android="http://schemas.android.com/apk/res/android"
      android:width="40dp"
      android:height="40dp"
      android:viewportWidth="24"
      android:viewportHeight="24">
  
          <path
              android:pathData="M19,13h-6v6h-2v-6H5v-2h6V5h2v6h6v2z"
              android:fillColor="#FFFFFF"/>
  
  </vector>

- En la carpeta `res`, crear una nueva carpeta de recusos con el nombre `layout`, botón secundario en la carpeta `res` y después click en `new -> Android Resource Directory`.<br>
Poner el nombre `layout` y seleccionar como `Resource type` = `layout`
- En la carpeta res > layout agregar los siguientes 4 archivos:
  - activity_product_list.xml
    ```xml activity_product_list.xml
    <?xml version="1.0" encoding="utf-8"?>
    <androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:id="@+id/main"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
    
        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/product_list"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            tools:listitem="@layout/item_list_product"/>
    
        <com.google.android.material.floatingactionbutton.FloatingActionButton
            android:id="@+id/button_add"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintBottom_toBottomOf="parent"
            android:layout_margin="24dp"
            android:backgroundTint="@color/teal_700"
            android:src="@drawable/ic_add"
            app:tint="@color/white" />
    
    </androidx.constraintlayout.widget.ConstraintLayout>
    
  - activity_product_detail.xml
    ``` xml activity_product_detail.xml
    <?xml version="1.0" encoding="utf-8"?>
    <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        xmlns:tools="http://schemas.android.com/tools">

      <TextView
          android:id="@+id/product_name"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:layout_centerHorizontal="true"
          android:layout_marginTop="24dp"
          android:textSize="24sp"
          android:textColor="@color/black"
          android:textStyle="bold"
          android:text="Product Name"/>
  
      <TextView
          android:id="@+id/product_price"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:layout_centerHorizontal="true"
          android:layout_below="@id/product_name"
          android:layout_marginTop="16dp"
          android:textSize="20sp"
          android:text="$0.00"/>
  
      <LinearLayout
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:layout_below="@+id/product_price"
          android:layout_marginTop="16dp"
          android:orientation="horizontal"
          android:gravity="center">
  
          <Button
              android:id="@+id/button_edit"
              android:layout_width="160dp"
              android:layout_height="wrap_content"
              android:text="Editar"/>
  
          <Button
              android:id="@+id/button_delete"
              android:layout_width="160dp"
              android:layout_height="wrap_content"
              android:layout_marginLeft="16dp"
              android:backgroundTint="#FF0000"
              android:text="Eliminar"/>
  
      </LinearLayout>

    </RelativeLayout>
  - activity_product_form.xml
    ```xml activity_product_form.xml
    <?xml version="1.0" encoding="utf-8"?>
    <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <EditText
            android:id="@+id/product_name"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_centerHorizontal="true"
            android:layout_margin="24dp"
            android:hint="Product Name"
            android:textColor="@color/black"
            android:textSize="24sp"
            android:textStyle="bold" />
    
        <LinearLayout
            android:id="@+id/product_price_container"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_below="@id/product_name"
            android:layout_centerHorizontal="true"
            android:layout_marginHorizontal="24dp"
            android:orientation="horizontal">
    
            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:textSize="20sp"
                android:text="$"/>
    
            <EditText
                android:id="@+id/product_price"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:hint="0.00"
                android:textSize="20sp" />
    
        </LinearLayout>
    
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_below="@+id/product_price_container"
            android:layout_marginTop="24dp"
            android:gravity="center"
            android:orientation="horizontal">
    
            <Button
                android:id="@+id/button_cancel"
                android:layout_width="160dp"
                android:layout_height="wrap_content"
                android:text="Cancelar" />
    
            <Button
                android:id="@+id/button_accept"
                android:layout_width="160dp"
                android:layout_height="wrap_content"
                android:layout_marginLeft="16dp"
                android:backgroundTint="#036D11"
                android:text="Aceptar" />
    
        </LinearLayout>

    </RelativeLayout>

  - item_list_product.xml
    ``` xml item_list.xml
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        xmlns:tools="http://schemas.android.com/tools"
        android:orientation="vertical"
        android:padding="12dp">
    
        <TextView
            android:id="@+id/product_name"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textColor="@color/black"
            android:textStyle="bold"
            android:textSize="18sp"
            tools:text="Nombre de producto"/>
    
        <TextView
            android:id="@+id/product_price"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginVertical="8dp"
            tools:text="$0.00"/>
    
        <View
            android:layout_width="match_parent"
            android:layout_height="1dp"
            android:background="#BBBBBB"/>
    
    </LinearLayout>

- Para continuar, crear un nuevo paquete con el nombre `ui` y crear las clases `ProductListActivity.java`, `ProductDetailActivity.java` y `ProductFormActivity.java`, estos corresponden a cada una de las pantallas que tendrá nuestra app, a continuación se muestra el códido de cada clase:
  - ProductListActivity.java
    ```Java ProductListActivity.java
    public class ProductListActivity extends AppCompatActivity implements ProductListContract.View, ProductListAdapter.OnItemSelectedListener, View.OnClickListener {

        ProductListPresenter presenter = new ProductListPresenter(this, RetrofitProvider.getRetrofit().create(ProductService.class));
        ActivityProductListBinding binding;
        List<Product> products = new ArrayList<>();
        ProductListAdapter adapter = new ProductListAdapter(products, this);
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            binding = ActivityProductListBinding.inflate(getLayoutInflater());
            binding.productList.setLayoutManager(new LinearLayoutManager(this, LinearLayoutManager.VERTICAL, false));
            binding.productList.setAdapter(adapter);
            binding.buttonAdd.setOnClickListener(this);
            setContentView(binding.getRoot());
        }
    
        @Override
        protected void onResume() {
            super.onResume();
            presenter.getProductList();
        }
    
        @Override
        public void showProducts(List<Product> products) {
            this.products.clear();
            this.products.addAll(products);
            binding.productList.setAdapter(adapter);
            adapter.notifyDataSetChanged();
    
        }
    
        @Override
        public void onItemSelected(View v, Product product) {
            Intent intent = new Intent(this, ProductDetailActivity.class);
            intent.putExtra(ProductDetailActivity.EXTRA_PRODUCT, product);
            startActivity(intent);
        }
    
        @Override
        public void onClick(View v) {
            if (v == binding.buttonAdd) {
                Intent intent = new Intent(this, ProductFormActivity.class);
                startActivity(intent);
            }
        }
    }
    
  - ProductDetailActivity.java
    ``` Java ProductDetailActivity.java
    public class ProductDetailActivity extends AppCompatActivity implements View.OnClickListener, ProductDetailContract.View {

        public static final String EXTRA_PRODUCT = "EXTRA_PRODUCT";
    
        ActivityProductDetailBinding binding;
        ProductDetailPresenter presenter = new ProductDetailPresenter(this, RetrofitProvider.getRetrofit().create(ProductService.class));
        Product product;
    
        @Override
        protected void onCreate(@Nullable Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            binding = ActivityProductDetailBinding.inflate(getLayoutInflater());
            setContentView(binding.getRoot());
    
            setTitle("Detalle");
            getSupportActionBar().setDisplayHomeAsUpEnabled(true);
    
            product = (Product) getIntent().getSerializableExtra(EXTRA_PRODUCT);
            binding.productName.setText(product.getName());
            binding.productPrice.setText("$" + product.getPrice());
            binding.buttonDelete.setOnClickListener(this);
            binding.buttonEdit.setOnClickListener(this);
        }
    
        @Override
        public boolean onOptionsItemSelected(MenuItem item) {
            switch (item.getItemId()) {
                case android.R.id.home:
                    getOnBackPressedDispatcher().onBackPressed();
                    return true;
                default:
                    return super.onOptionsItemSelected(item);
            }
        }
    
        @Override
        public void onClick(View v) {
            if (v == binding.buttonDelete) {
                presenter.deleteProduct(product.getId());
            } else if (v == binding.buttonEdit) {
                Intent intent = new Intent(this, ProductFormActivity.class);
                intent.putExtra(ProductFormActivity.EXTRA_PRODUCT, product);
                startActivity(intent);
            }
        }
    
        @Override
        public void onProductDeleted() {
            finish();
        }
    }
  - ProductFormActivity.java
    ```Java ProductFormActivity.java
    public class ProductFormActivity extends AppCompatActivity implements View.OnClickListener, ProductFormContract.View {
        public static final String EXTRA_PRODUCT = "EXTRA_PRODUCT";
    
        ActivityProductFormBinding binding;
        Product product;
        ProductFormPresenter presenter = new ProductFormPresenter(this, RetrofitProvider.getRetrofit().create(ProductService.class));
    
        @Override
        protected void onCreate(@Nullable Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            binding = ActivityProductFormBinding.inflate(getLayoutInflater());
            setContentView(binding.getRoot());
    
            product = (Product) getIntent().getSerializableExtra(EXTRA_PRODUCT);
            if (isInEditMode()) {
                binding.productName.setText(product.getName());
                binding.productPrice.setText(product.getPrice());
            }
            binding.buttonAccept.setOnClickListener(this);
            binding.buttonCancel.setOnClickListener(this);
    
            setTitle(isInEditMode() ? "Editar" : "Crear");
            getSupportActionBar().setDisplayHomeAsUpEnabled(true);
        }
    
        @Override
        public boolean onOptionsItemSelected(MenuItem item) {
            switch (item.getItemId()) {
                case android.R.id.home:
                    getOnBackPressedDispatcher().onBackPressed();
                    return true;
                default:
                    return super.onOptionsItemSelected(item);
            }
        }
    
        @Override
        public void onClick(View v) {
            if (v == binding.buttonCancel) {
                finish();
            } else if (v == binding.buttonAccept) {
                if (isInEditMode()) {
                    Product updatedProduct = new Product();
                    updatedProduct.setName(binding.productName.getText().toString());
                    updatedProduct.setPrice(binding.productPrice.getText().toString());
                    presenter.updateProduct(product.getId(), updatedProduct);
                } else {
                    Product productToCreate = new Product();
                    productToCreate.setName(binding.productName.getText().toString());
                    productToCreate.setPrice(binding.productPrice.getText().toString());
                    presenter.createProduct(productToCreate);
                }
            }
        }
    
        private boolean isInEditMode() {
            return product != null;
        }
    
        @Override
        public void onProductUpdated() {
            goToProductList();
        }
    
        @Override
        public void onProductCreated() {
            goToProductList();
        }
    
        private void goToProductList() {
            Intent intent = new Intent(this, ProductListActivity.class);
            intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK);
            startActivity(intent);
            finish();
        }
    }
- En este punto, la clase ProductListActivity muestra un error porque hay la clase `ProductListAdapter` no existe aun, para solucionarlo, se crea 2 clases nuevas en el paquete ui, la primera es `ProductViewHolder` y la segunda es `ProductListAdapter`. Estas sirven para mostrar la lista de productos una vez que son descargados de la API, a continuación el código de estas clases
  - ProductViewHolder.java
    ```Java ProductViewHolder.java
    public class ProductViewHolder extends RecyclerView.ViewHolder {
        ItemListProductBinding binding;
    
        public ProductViewHolder(@NonNull View itemView) {
            super(itemView);
            binding = ItemListProductBinding.bind(itemView);
        }
    }
  - ProductListAdapter.java
    ```Java ProductListAdapter.java
    public class ProductListAdapter extends RecyclerView.Adapter<ProductViewHolder> {

        List<Product> items;
        OnItemSelectedListener listener;
    
        public ProductListAdapter(List<Product> items) {
            this.items = items;
        }
    
        public ProductListAdapter(List<Product> items, OnItemSelectedListener listener) {
            this.items = items;
            this.listener = listener;
        }
    
        @NonNull
        @Override
        public ProductViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
            LayoutInflater inflater = LayoutInflater.from(parent.getContext());
            ProductViewHolder vh = new ProductViewHolder(inflater.inflate(R.layout.item_list_product, parent, false));
            return vh;
        }
    
        @Override
        public void onBindViewHolder(@NonNull ProductViewHolder holder, int position) {
            Product product = items.get(position);
            holder.binding.productName.setText(product.getName());
            holder.binding.productPrice.setText("$"+product.getPrice());
            holder.itemView.setOnClickListener((v)-> {
                if (listener != null) {
                    listener.onItemSelected(holder.itemView, product);
                }
            });
        }
    
        @Override
        public int getItemCount() {
            return items == null ? 0 : items.size();
        }
    
        public void setListener(OnItemSelectedListener listener) {
            this.listener = listener;
        }
    
        public interface OnItemSelectedListener {
            void onItemSelected(View item, Product product);
        }
    }
- Para finalizar, en al archivo `Manifest.xml` se define que la app usará internet y tambien las pantalas que tendrá la app. Queda de la siguiente manera:
  ```xml Manifest.xml
  <?xml version="1.0" encoding="utf-8"?>
  <manifest xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:tools="http://schemas.android.com/tools">
  
      <uses-permission android:name="android.permission.INTERNET" />
  
      <application
          android:allowBackup="true"
          android:dataExtractionRules="@xml/data_extraction_rules"
          android:fullBackupContent="@xml/backup_rules"
          android:icon="@mipmap/ic_launcher"
          android:label="@string/app_name"
          android:roundIcon="@mipmap/ic_launcher_round"
          android:supportsRtl="true"
          android:theme="@style/Theme.AppCompat"
          tools:targetApi="31">
          <activity
              android:name=".ui.ProductListActivity"
              android:exported="true" >
              <intent-filter>
                  <action android:name="android.intent.action.MAIN" />
                  <action android:name="android.intent.action.VIEW" />
  
                  <category android:name="android.intent.category.LAUNCHER" />
              </intent-filter>
          </activity>
  
          <activity
              android:name=".ui.ProductDetailActivity"
              android:exported="false" >
          </activity>
  
          <activity
              android:name=".ui.ProductFormActivity"
              android:exported="false" >
          </activity>
      </application>
  
  </manifest>


