# roomczolko

implementation "androidx.room:room-runtime:2.6.1"
    annotationProcessor "androidx.room:room-compiler:2.6.1"

<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/kategoriaTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="50dp"
        android:textSize="24sp"
        android:text="Kategoria"
        android:gravity="center"/>

    <!-- Widok tekstowy do wyświetlania słowa -->
    <TextView
        android:id="@+id/slowoTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/kategoriaTextView"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="30dp"
        android:textSize="30sp"
        android:text="Słowo"
        android:gravity="center"/>

    <!-- Przycisk do losowania kategorii -->
    <Button
        android:id="@+id/przyciskKategoria"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/slowoTextView"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="30dp"
        android:text="Wybierz kategorię" />

    <!-- Przycisk do losowania słowa -->
    <Button
        android:id="@+id/przyciskSlowo"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/przyciskKategoria"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="20dp"
        android:text="Następne słowo" />

    <!-- Spinner do wyboru kategorii -->
    <Spinner
        android:id="@+id/kategoriaSpinner"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/przyciskSlowo"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="30dp" />

    <!-- Pole do wpisania nowego słowa -->
    <EditText
        android:id="@+id/noweSlowoEditText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/kategoriaSpinner"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="20dp"
        android:hint="Wpisz nowe słowo"
        android:inputType="text" />

    <!-- Przycisk do dodawania nowego słowa -->
    <Button
        android:id="@+id/przyciskDodajSlowo"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/noweSlowoEditText"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="20dp"
        android:text="Dodaj słowo" />
</RelativeLayout>


import android.os.Bundle;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Spinner;
import android.widget.TextView;
import android.widget.Toast;

import androidx.appcompat.app.AppCompatActivity;

import java.util.List;
import java.util.Random;

public class MainActivity extends AppCompatActivity {

    private AppDatabase db;
    private List<String> kategorie;
    private List<String> slowa;
    private String aktualnaKategoria;

    private TextView kategoriaTextView;
    private TextView slowoTextView;
    private Button przyciskKategoria;
    private Button przyciskSlowo;
    private Spinner kategoriaSpinner;
    private EditText noweSlowoEditText;
    private Button przyciskDodajSlowo;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        db = AppDatabase.getDatabase(getApplicationContext());
        DatabaseInitializer.populateAsync(db);

        kategoriaTextView = findViewById(R.id.kategoriaTextView);
        slowoTextView = findViewById(R.id.slowoTextView);
        przyciskKategoria = findViewById(R.id.przyciskKategoria);
        przyciskSlowo = findViewById(R.id.przyciskSlowo);
        kategoriaSpinner = findViewById(R.id.kategoriaSpinner);
        noweSlowoEditText = findViewById(R.id.noweSlowoEditText);
        przyciskDodajSlowo = findViewById(R.id.przyciskDodajSlowo);

        przyciskKategoria.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                wylosujKategorie();
            }
        });

        przyciskSlowo.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                wylosujSlowo();
            }
        });

        przyciskDodajSlowo.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                dodajNoweSlowo();
            }
        });

        załadujKategorieDoSpinnera();
    }

    private void wylosujKategorie() {
        kategorie = db.kategoriaDao().getAllCategories();
        if (!kategorie.isEmpty()) {
            Random random = new Random();
            aktualnaKategoria = kategorie.get(random.nextInt(kategorie.size()));
            kategoriaTextView.setText(aktualnaKategoria);
            wylosujSlowo();
        }
    }

    private void wylosujSlowo() {
        if (aktualnaKategoria != null) {
            int kategoriaId = kategorie.indexOf(aktualnaKategoria) + 1;
            slowa = db.slowoDao().getWordsForCategory(kategoriaId);
            if (!slowa.isEmpty()) {
                Random random = new Random();
                String slowo = slowa.get(random.nextInt(slowa.size()));
                slowoTextView.setText(slowo);
            }
        }
    }

    private void załadujKategorieDoSpinnera() {
        kategorie = db.kategoriaDao().getAllCategories();
        ArrayAdapter<String> adapter = new ArrayAdapter<>(this, android.R.layout.simple_spinner_item, kategorie);
        adapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
        kategoriaSpinner.setAdapter(adapter);
    }

    private void dodajNoweSlowo() {
        String noweSlowo = noweSlowoEditText.getText().toString().trim();
        String wybranaKategoria = kategoriaSpinner.getSelectedItem().toString();

        if (!noweSlowo.isEmpty() && wybranaKategoria != null) {
            int kategoriaId = kategorie.indexOf(wybranaKategoria) + 1;
            Slowo slowo = new Slowo(noweSlowo, kategoriaId);
            db.slowoDao().insert(slowo);

            Toast.makeText(this, "Dodano słowo: " + noweSlowo + " do kategorii: " + wybranaKategoria, Toast.LENGTH_SHORT).show();
            noweSlowoEditText.setText("");
        } else {
            Toast.makeText(this, "Wprowadź słowo i wybierz kategorię.", Toast.LENGTH_SHORT).show();
        }
    }
}


import android.content.Context;

import androidx.room.Database;
import androidx.room.Room;
import androidx.room.RoomDatabase;

@Database(entities = {Kategoria.class, Slowo.class}, version = 1)
public abstract class AppDatabase extends RoomDatabase {
    public abstract KategoriaDao kategoriaDao();
    public abstract SlowoDao slowoDao();

    private static volatile AppDatabase INSTANCE;

    static AppDatabase getDatabase(final Context context) {
        if (INSTANCE == null) {
            synchronized (AppDatabase.class) {
                if (INSTANCE == null) {
                    INSTANCE = Room.databaseBuilder(context.getApplicationContext(),
                                    AppDatabase.class, "czolko_database")
                            .allowMainThreadQueries()
                            .build();
                }
            }
        }
        return INSTANCE;
    }
}




import androidx.room.Entity;
import androidx.room.PrimaryKey;

@Entity
public class Kategoria {
    @PrimaryKey(autoGenerate = true)
    public int id;

    public String nazwa;

    public Kategoria(String nazwa) {
        this.nazwa = nazwa;
    }
}



import androidx.room.Entity;
import androidx.room.PrimaryKey;

@Entity
public class Slowo {
    @PrimaryKey(autoGenerate = true)
    public int id;

    public String nazwa;
    public int kategoriaId;

    public Slowo(String nazwa, int kategoriaId) {
        this.nazwa = nazwa;
        this.kategoriaId = kategoriaId;
    }
}




import androidx.room.Dao;
import androidx.room.Insert;
import androidx.room.Query;

import java.util.List;

@Dao
public interface KategoriaDao {
    @Query("SELECT nazwa FROM Kategoria")
    List<String> getAllCategories();

    @Insert
    void insert(Kategoria kategoria);
}




import androidx.room.Dao;
import androidx.room.Insert;
import androidx.room.Query;

import java.util.List;

@Dao
public interface SlowoDao {
    @Query("SELECT nazwa FROM Slowo WHERE kategoriaId = :kategoriaId")
    List<String> getWordsForCategory(int kategoriaId);

    @Insert
    void insert(Slowo slowo);
}




import java.util.concurrent.Executors;

public class DatabaseInitializer {

    public static void populateAsync(final AppDatabase db) {
        Executors.newSingleThreadExecutor().execute(() -> populateSync(db));
    }

    private static void populateSync(AppDatabase db) {
        KategoriaDao kategoriaDao = db.kategoriaDao();
        SlowoDao slowoDao = db.slowoDao();

        if (kategoriaDao.getAllCategories().isEmpty()) {
            Kategoria k1 = new Kategoria("Sport");
            Kategoria k2 = new Kategoria("Muzyka");
            Kategoria k3 = new Kategoria("Filmy");
            Kategoria k4 = new Kategoria("Jedzenie");

            kategoriaDao.insert(k1);
            kategoriaDao.insert(k2);
            kategoriaDao.insert(k3);
            kategoriaDao.insert(k4);

            slowoDao.insert(new Slowo("Piłka", 1));
            slowoDao.insert(new Slowo("Gitara", 2));
            slowoDao.insert(new Slowo("Aktor", 3));
            slowoDao.insert(new Slowo("Pizza", 4));
        }
    }
}
