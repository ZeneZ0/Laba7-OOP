# Laba7-OOP
MainActivity:
package com.example.laba7;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;
import android.text.Editable;
import android.text.TextWatcher;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.RadioButton;
import android.widget.RadioGroup;
import android.widget.TextView;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {
    private EditText numberInput;
    private RadioGroup baseGroup;
    private RadioButton base2, base8, base10, base16, baseCustom;
    private EditText customBaseInput;
    private Button convertButton;
    private TextView binaryResult, octalResult, decimalResult, hexResult;
    private TextView detectionResult;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initViews();
        setupListeners();
    }

    private void initViews() {
        numberInput = findViewById(R.id.numberInput);
        baseGroup = findViewById(R.id.baseGroup);
        base2 = findViewById(R.id.base2);
        base8 = findViewById(R.id.base8);
        base10 = findViewById(R.id.base10);
        base16 = findViewById(R.id.base16);
        baseCustom = findViewById(R.id.baseCustom);
        customBaseInput = findViewById(R.id.customBaseInput);
        convertButton = findViewById(R.id.convertButton);
        binaryResult = findViewById(R.id.binaryResult);
        octalResult = findViewById(R.id.octalResult);
        decimalResult = findViewById(R.id.decimalResult);
        hexResult = findViewById(R.id.hexResult);
        detectionResult = findViewById(R.id.detectionResult);
    }

    private void setupListeners() {
        baseGroup.setOnCheckedChangeListener((group, checkedId) -> {
            customBaseInput.setVisibility(checkedId == R.id.baseCustom ? View.VISIBLE : View.GONE);
        });

        convertButton.setOnClickListener(v -> convertNumber());

        numberInput.addTextChangedListener(new TextWatcher() {
            @Override public void beforeTextChanged(CharSequence s, int start, int count, int after) {}
            @Override public void afterTextChanged(Editable s) {}

            @Override
            public void onTextChanged(CharSequence s, int start, int before, int count) {
                detectionResult.setText(NumberConverter.detectPossibleBases(s.toString()));
            }
        });
    }

    private void convertNumber() {
        String numberStr = numberInput.getText().toString().trim();
        if (numberStr.isEmpty()) {
            Toast.makeText(this, "Введите число", Toast.LENGTH_SHORT).show();
            return;
        }

        try {
            int fromBase = getSelectedBase();

            binaryResult.setText("Двоичная: " + NumberConverter.convertFromBase(numberStr, fromBase, 2));
            octalResult.setText("Восьмеричная: " + NumberConverter.convertFromBase(numberStr, fromBase, 8));
            decimalResult.setText("Десятичная: " + NumberConverter.convertFromBase(numberStr, fromBase, 10));
            hexResult.setText("Шестнадцатеричная: " +
                    NumberConverter.convertFromBase(numberStr, fromBase, 16).toUpperCase());

        } catch (NumberFormatException e) {
            Toast.makeText(this, "Некорректное число для выбранной системы", Toast.LENGTH_SHORT).show();
            clearResults();
        }
    }

    private int getSelectedBase() {
        if (base2.isChecked()) return 2;
        if (base8.isChecked()) return 8;
        if (base10.isChecked()) return 10;
        if (base16.isChecked()) return 16;
        if (baseCustom.isChecked()) {
            try {
                int customBase = Integer.parseInt(customBaseInput.getText().toString());
                if (customBase < 1 || customBase > 10) {
                    Toast.makeText(this, "Основание должно быть от 1 до 10", Toast.LENGTH_SHORT).show();
                    return 10;
                }
                return customBase;
            } catch (NumberFormatException e) {
                Toast.makeText(this, "Введите корректное основание", Toast.LENGTH_SHORT).show();
                return 10;
            }
        }
        return 10;
    }

    private void clearResults() {
        binaryResult.setText("");
        octalResult.setText("");
        decimalResult.setText("");
        hexResult.setText("");
    }
}
NumberConverter:
package com.example.laba7;

public class NumberConverter {

    // Проверка, подходит ли число для указанной системы счисления
    public static boolean isValidForBase(String numberStr, int base) {
        try {
            if (base == 16) {
                return numberStr.matches("[0-9A-Fa-f]+");
            } else {
                for (char c : numberStr.toCharArray()) {
                    if (Character.digit(c, base) == -1) {
                        return false;
                    }
                }
                return true;
            }
        } catch (Exception e) {
            return false;
        }
    }

    // Конвертация числа из одной системы в другую
    public static String convertFromBase(String numberStr, int fromBase, int toBase)
            throws NumberFormatException {
        long decimalValue = Long.parseLong(numberStr, fromBase);
        return Long.toString(decimalValue, toBase);
    }

    // Определение возможных систем счисления для введенного числа
    public static String detectPossibleBases(String numberStr) {
        if (numberStr.isEmpty()) {
            return "";
        }

        StringBuilder detectionMessage = new StringBuilder("Возможные системы: ");

        if (isValidForBase(numberStr, 2)) detectionMessage.append("2, ");
        if (isValidForBase(numberStr, 8)) detectionMessage.append("8, ");
        if (isValidForBase(numberStr, 10)) detectionMessage.append("10, ");
        if (isValidForBase(numberStr, 16)) detectionMessage.append("16, ");

        if (detectionMessage.length() > "Возможные системы: ".length()) {
            detectionMessage.setLength(detectionMessage.length() - 2);
            return detectionMessage.toString();
        }
        return "Не соответствует ни одной системе";
    }
}
