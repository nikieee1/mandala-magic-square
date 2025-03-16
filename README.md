# mandala-magic-square
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Магический квадрат</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background: linear-gradient(135deg, #f5f7fa, #c3cfe2);
            margin: 0;
        }
        .container {
            background: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
            text-align: center;
        }
        h1 {
            color: #333;
        }
        form {
            margin-bottom: 20px;
        }
        input[type="number"], input[type="text"] {
            padding: 10px;
            margin: 5px;
            border: 1px solid #ccc;
            border-radius: 5px;
        }
        input[type="submit"], button {
            padding: 10px 20px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            margin: 5px;
        }
        input[type="submit"]:hover, button:hover {
            background-color: #45a049;
        }
        .magic-square {
            display: inline-grid;
            grid-template-columns: repeat(3, 70px);
            gap: 8px;
            margin-top: 20px;
        }
        .cell {
            width: 70px;
            height: 70px;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 28px;
            font-weight: bold;
            border-radius: 8px;
            border: 2px solid #ddd;
            animation: fadeIn 0.5s ease-in forwards;
            opacity: 0;
        }
        @keyframes fadeIn {
            0% { opacity: 0; transform: scale(0.5); }
            100% { opacity: 1; transform: scale(1); }
        }
        .cell:nth-child(1) { animation-delay: 0.1s; }
        .cell:nth-child(2) { animation-delay: 0.2s; }
        .cell:nth-child(3) { animation-delay: 0.3s; }
        .cell:nth-child(4) { animation-delay: 0.4s; }
        .cell:nth-child(5) { animation-delay: 0.5s; }
        .cell:nth-child(6) { animation-delay: 0.6s; }
        .cell:nth-child(7) { animation-delay: 0.7s; }
        .cell:nth-child(8) { animation-delay: 0.8s; }
        .cell:nth-child(9) { animation-delay: 0.9s; }
        .highlight {
            border-color: #000;
        }
        .message {
            margin-top: 20px;
            font-size: 18px;
            color: #555;
        }
        .error {
            color: #ff0000;
            font-weight: bold;
        }
        .palette {
            margin-top: 20px;
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 10px;
        }
        .palette-item {
            width: 50px;
            height: 50px;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 18px;
            font-weight: bold;
            border-radius: 5px;
            border: 1px solid #ccc;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Создай свой магический квадрат</h1>
        <form method="post">
            <label>Введите число (1–9):</label><br>
            <input type="number" name="number" min="1" max="9" value="<?php echo isset($_POST['number']) ? htmlspecialchars($_POST['number']) : ''; ?>"><br>
            <label>Или введите слово:</label><br>
            <input type="text" name="word" value="<?php echo isset($_POST['word']) ? htmlspecialchars($_POST['word']) : ''; ?>"><br>
            <input type="submit" name="generate" value="Сгенерировать">
        </form>

        <?php
        if ($_SERVER["REQUEST_METHOD"] == "POST" && (isset($_POST['generate']) || isset($_POST['refresh']))) {
            // Базовая палитра (фиксированная)
            $baseColors = [
                0 => '#ffffff', // белый (не используется в шкале)
                1 => '#ff0000', // красный
                2 => '#0000ff', // темно-синий
                3 => '#00ff00', // зелёный
                4 => '#ffff00', // жёлтый
                5 => '#0080ff', // светло-голубой
                6 => '#00ffff', // бирюзовый
                7 => '#ff00ff', // розовый
                8 => '#ff8000', // оранжевый
                9 => '#800080'  // фиолетовый
            ];

            function getShiftedPalette($c, $baseColors) {
                $shiftedColors = [];
                $shiftedColors[0] = $baseColors[0]; // 0 всегда белый (для ячеек)

                // Сдвигаем палитру вправо в зависимости от c
                for ($i = 1; $i <= 9; $i++) {
                    $shift = ($c - 1); // Сдвиг на (c-1) шагов
                    $baseIndex = (($i - 1 + $shift) % 9) + 1; // Циклический сдвиг вправо
                    if ($baseIndex == 0) $baseIndex = 9; // Корректируем
                    $shiftedColors[$i] = $baseColors[$baseIndex];
                }

                return $shiftedColors;
            }

            function calculateMagicSquare($c) {
                $c = max(1, min(9, $c));

                // Магическая константа
                $S = 3 * $c;

                // Выбираем b так, чтобы все числа были в диапазоне 1–9
                $b_min = max(1, 2 * $c - 9); // 2c - b >= 1 => b <= 2c - 1
                $b_max = min(9, 2 * $c - 1); // 2c - b <= 9 => b >= 2c - 9
                if ($b_min > $b_max) {
                    return [null, null]; // Невозможно построить квадрат
                }

                $b = rand($b_min, $b_max);
                $a = 2 * $c - $b;
                $f = 2 * $c - $b;
                $h = 2 * $c - $b;

                // Формируем квадрат
                $target = [
                    [$a, $b, $c],
                    [$b, $c, $f],
                    [$c, $h, $b]
                ];

                // Проверяем
                $rowSums = array_map('array_sum', $target);
                $colSums = array_map(function($col) use ($target) {
                    return array_sum(array_column($target, $col));
                }, range(0, 2));
                $diag1 = $target[0][0] + $target[1][1] + $target[2][2];
                $diag2 = $target[0][2] + $target[1][1] + $target[2][0];

                $isMagic = ($rowSums[0] == $S && $rowSums[1] == $S && $rowSums[2] == $S &&
                            $colSums[0] == $S && $colSums[1] == $S && $colSums[2] == $S &&
                            $diag1 == $S && $diag2 == $S);

                if (!$isMagic) {
                    return [null, null];
                }

                return [$target, $S];
            }

            function wordToNumber($word) {
                $word = strtolower($word);
                $sum = 0;
                for ($i = 0; $i < strlen($word); $i++) {
                    if (ctype_alpha($word[$i])) {
                        $sum += ord($word[$i]) - ord('a') + 1;
                    }
                }
                while ($sum > 9) {
                    $sum = array_sum(str_split($sum));
                }
                return $sum ?: 1;
            }

            $inputNumber = isset($_POST["number"]) && !empty($_POST["number"]) ? (int)$_POST["number"] : null;
            $inputWord = isset($_POST["word"]) && !empty($_POST["word"]) ? trim($_POST["word"]) : null;

            if ($inputNumber === null && $inputWord === null) {
                $c = 1;
                echo "<div class='message'>Не введено число или слово, использовано значение по умолчанию: $c</div>";
            } elseif ($inputNumber !== null) {
                $c = $inputNumber;
                echo "<div class='message'>Вы ввели число: $c</div>";
            } elseif ($inputWord) {
                $c = wordToNumber($inputWord);
                echo "<div class='message'>Ваше слово '$inputWord' преобразовано в число: $c</div>";
            }

            // Получаем сдвинутую палитру
            $colors = getShiftedPalette($c, $baseColors);

            list($magicSquare, $magicConstant) = calculateMagicSquare($c);

            if ($magicSquare) {
                echo "<div class='magic-square'>";
                for ($i = 0; $i < 3; $i++) {
                    for ($j = 0; $j < 3; $j++) {
                        $number = $magicSquare[$i][$j];
                        $colorIndex = $number; // Используем сдвинутую палитру
                        $class = ($i == 2 && $j == 0) ? "cell highlight" : "cell";
                        $color = $colors[$colorIndex];
                        // Определяем цвет текста в зависимости от фона
                        $textColor = in_array($colorIndex, [1, 2, 7, 9]) ? '#fff' : '#000';
                        echo "<div class='$class' style='background-color: $color; color: $textColor;'>$number</div>";
                    }
                }
                echo "</div>";
                echo "<div class='message'>Магическая константа: $magicConstant</div>";

                // Выводим шкалу палитры (без 0)
                echo "<div class='message'>Шкала палитры:</div>";
                echo "<div class='palette'>";
                for ($num = 1; $num <= 9; $num++) {
                    $color = $colors[$num];
                    $textColor = in_array($num, [1, 2, 7, 9]) ? '#fff' : '#000';
                    echo "<div class='palette-item' style='background-color: $color; color: $textColor;'>$num</div>";
                }
                echo "</div>";
            } else {
                echo "<div class='error'>Невозможно построить магический квадрат для c = $c с числами от 1 до 9. Попробуйте другое число.</div>";
            }

            echo "<form method='post'>";
            echo "<input type='hidden' name='number' value='" . htmlspecialchars($c) . "'>";
            echo "<button type='submit' name='refresh'>Обновить матрицу</button>";
            echo "</form>";
        }
        ?>
    </div>
</body>
</html>
