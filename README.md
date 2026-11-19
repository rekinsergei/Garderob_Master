
package com.example.my_gorderob

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.*
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.style.TextOverflow
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import java.util.Calendar

private val Bg = Color(0xFF090909)
private val Panel = Color(0xFF1C1C1C)
private val Cyan = Color(0xFF22D3EE)
private val Muted = Color(0xFF9CA3AF)
private val Green = Color(0xFF67D391)
private val Pink = Color(0xFFF472B6)

data class Clothing(
    val id: Int,
    val name: String,
    val category: String,
    val material: String,
    val symbol: String,
    val condition: Float = 0.85f,
    val needsCare: Boolean = false
)

data class CareRecord(
    val clothingId: Int,
    val day: Int,
    val month: Int,
    val year: Int,
    val procedure: String
)

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MaterialTheme {
                GarderobApp()
            }
        }
    }
}

@Composable
fun GarderobApp() {
    val today = remember { Calendar.getInstance() }

    var page by remember { mutableStateOf("Вещи") }
    var filter by remember { mutableStateOf("Все") }
    var name by remember { mutableStateOf("") }
    var category by remember { mutableStateOf("Обувь") }
    var material by remember { mutableStateOf("") }
    var selectedId by remember { mutableIntStateOf(1) }
    var procedure by remember { mutableStateOf("Чистка") }

    var selectedDay by remember {
        mutableIntStateOf(today.get(Calendar.DAY_OF_MONTH))
    }
    var selectedMonth by remember {
        mutableIntStateOf(today.get(Calendar.MONTH))
    }
    var selectedYear by remember {
        mutableIntStateOf(today.get(Calendar.YEAR))
    }

    var displayMonth by remember {
        mutableIntStateOf(today.get(Calendar.MONTH))
    }
    var displayYear by remember {
        mutableIntStateOf(today.get(Calendar.YEAR))
    }

    var reminders by remember { mutableStateOf(true) }
    var tips by remember { mutableStateOf(true) }
    var news by remember { mutableStateOf(false) }

    val clothes = remember {
        mutableStateListOf(
            Clothing(1, "Nike Air Jordan 1", "Обувь", "Кожа", "👟"),
            Clothing(2, "Ботинки Chelsea", "Обувь", "Замша", "🥾", 0.72f, true),
            Clothing(3, "Пальто шерстяное", "Одежда", "Шерсть", "🧥"),
            Clothing(4, "Кроссовки Run", "Обувь", "Текстиль", "👟", 0.64f, true),
            Clothing(5, "Белая рубашка", "Одежда", "Хлопок", "👔"),
            Clothing(6, "Сумка", "Аксессуары", "Кожа", "👜", 0.68f, true)
        )
    }

    val records = remember {
        mutableStateListOf<CareRecord>()
    }

    val selectedItem = clothes.find { it.id == selectedId }
    val visible = clothes.filter {
        filter == "Все" || it.category == filter
    }

    Scaffold(
        containerColor = Bg,
        bottomBar = {
            NavigationBar(containerColor = Panel) {
                val tabs = listOf(
                    "Вещи", "Календарь", "Статистика",
                    "Справочник", "Настройки"
                )
                val icons = listOf("▦", "▤", "⌁", "✧", "⚙")

                tabs.forEachIndexed { index, tab ->
                    NavigationBarItem(
                        selected = page == tab,
                        onClick = { page = tab },
                        icon = {
                            Text(
                                icons[index],
                                color = if (page == tab) Cyan else Muted,
                                fontSize = 19.sp
                            )
                        },
                        label = {
                            Text(tab, fontSize = 9.sp)
                        },
                        colors = NavigationBarItemDefaults.colors(
                            indicatorColor = Color(0xFF303030),
                            selectedTextColor = Color.White,
                            unselectedTextColor = Muted
                        )
                    )
                }
            }
        }
    ) { innerPadding ->
        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(innerPadding)
                .background(Bg)
                .verticalScroll(rememberScrollState())
        ) {
            AppHeader(
                onSettings = { page = "Настройки" }
            )

            Column(
                modifier = Modifier.padding(16.dp),
                verticalArrangement = Arrangement.spacedBy(16.dp)
            ) {
                when (page) {
                    "Вещи" -> {
                        SectionTitle("ТВОЙ ГАРДЕРОБ", "Моя коллекция")

                        Button(
                            onClick = { page = "Добавить" },
                            colors = ButtonDefaults.buttonColors(
                                containerColor = Color.White,
                                contentColor = Color.Black
                            )
                        ) {
                            Text("+ Добавить")
                        }

                        DarkCard {
                            WhiteText(
                                "Добро пожаловать в Гардероб Мастер",
                                14,
                                true
                            )
                            GrayText(
                                "Заботься о любимых вещах — следи " +
                                        "за чисткой, состоянием и планируй уход."
                            )
                        }

                        Row(
                            horizontalArrangement = Arrangement.spacedBy(8.dp)
                        ) {
                            StatBox(
                                clothes.size.toString(),
                                "Всего вещей",
                                Modifier.weight(1f)
                            )
                            StatBox(
                                clothes.count { it.needsCare }.toString(),
                                "Пора ухаживать",
                                Modifier.weight(1f)
                            )
                            StatBox(
                                records.size.toString(),
                                "Записей ухода",
                                Modifier.weight(1f)
                            )
                        }

                        Row(
                            modifier = Modifier.horizontalScroll(
                                rememberScrollState()
                            ),
                            horizontalArrangement = Arrangement.spacedBy(6.dp)
                        ) {
                            listOf(
                                "Все", "Обувь", "Одежда", "Аксессуары"
                            ).forEach { value ->
                                FilterChip(
                                    selected = filter == value,
                                    onClick = { filter = value },
                                    label = {
                                        Text(value, fontSize = 11.sp)
                                    }
                                )
                            }
                        }

                        WhiteText("Ваши вещи · ${visible.size}", 15, true)

                        visible.chunked(2).forEach { pair ->
                            Row(
                                horizontalArrangement = Arrangement.spacedBy(8.dp)
                            ) {
                                pair.forEach { item ->
                                    Box(Modifier.weight(1f)) {
                                        ClothingCard(
                                            item = item,
                                            onDetails = {
                                                selectedId = item.id
                                                page = "Подробнее"
                                            },
                                            onCare = {
                                                selectedId = item.id
                                                page = "Уход"
                                            }
                                        )
                                    }
                                }
                                if (pair.size == 1) {
                                    Spacer(Modifier.weight(1f))
                                }
                            }
                        }
                    }

                    "Добавить" -> {
                        SectionTitle("КОЛЛЕКЦИЯ", "Добавить вещь")

                        DarkField(name, { name = it }, "Название вещи")
                        WhiteText("Категория", 14, true)

                        listOf(
                            "Обувь", "Одежда", "Аксессуары"
                        ).forEach { value ->
                            Row(
                                verticalAlignment = Alignment.CenterVertically
                            ) {
                                RadioButton(
                                    selected = category == value,
                                    onClick = { category = value }
                                )
                                WhiteText(value, 14)
                            }
                        }

                        DarkField(
                            material,
                            { material = it },
                            "Материал"
                        )

                        Button(
                            onClick = {
                                if (name.isNotBlank()) {
                                    val nextId =
                                        (clothes.maxOfOrNull { it.id } ?: 0) + 1

                                    clothes.add(
                                        Clothing(
                                            nextId,
                                            name.trim(),
                                            category,
                                            material.trim(),
                                            when (category) {
                                                "Обувь" -> "👟"
                                                "Одежда" -> "👕"
                                                else -> "👜"
                                            }
                                        )
                                    )
                                    name = ""
                                    material = ""
                                    page = "Вещи"
                                }
                            },
                            enabled = name.isNotBlank()
                        ) {
                            Text("Сохранить вещь")
                        }

                        BackButton { page = "Вещи" }
                    }

                    "Подробнее" -> {
                        SectionTitle("МОЯ КОЛЛЕКЦИЯ", "Информация о вещи")

                        if (selectedItem != null) {
                            DarkCard {
                                WhiteText(selectedItem.symbol, 48)
                                WhiteText(selectedItem.name, 22, true)
                                GrayText(
                                    "${selectedItem.category} · " +
                                            selectedItem.material
                                )
                                WhiteText(
                                    "Состояние: " +
                                            "${(selectedItem.condition * 100).toInt()}%"
                                )
                                LinearProgressIndicator(
                                    progress = { selectedItem.condition },
                                    modifier = Modifier.fillMaxWidth(),
                                    color = Pink
                                )
                            }

                            Button(
                                onClick = { page = "Уход" }
                            ) {
                                Text("+ Записать уход")
                            }
                        }

                        BackButton { page = "Вещи" }
                    }

                    "Уход" -> {
                        SectionTitle("ИСТОРИЯ УХОДА", "Добавить уход")

                        WhiteText(
                            selectedItem?.name ?: "Вещь",
                            18,
                            true
                        )

                        WhiteText("Процедура", 14)

                        listOf(
                            "Чистка", "Стирка", "Обработка", "Ремонт"
                        ).forEach { value ->
                            Row(
                                verticalAlignment = Alignment.CenterVertically
                            ) {
                                RadioButton(
                                    selected = procedure == value,
                                    onClick = { procedure = value }
                                )
                                WhiteText(value, 14)
                            }
                        }

                        Button(
                            onClick = {
                                val now = Calendar.getInstance()
                                records.add(
                                    CareRecord(
                                        selectedId,
                                        now.get(Calendar.DAY_OF_MONTH),
                                        now.get(Calendar.MONTH),
                                        now.get(Calendar.YEAR),
                                        procedure
                                    )
                                )
                                page = "Вещи"
                            }
                        ) {
                            Text("Сохранить запись")
                        }

                        BackButton { page = "Вещи" }
                    }

                    "Календарь" -> {
                        SectionTitle("ПЛАНИРОВАНИЕ", "Календарь ухода")

                        val monthNames = listOf(
                            "январь", "февраль", "март", "апрель",
                            "май", "июнь", "июль", "август",
                            "сентябрь", "октябрь", "ноябрь", "декабрь"
                        )

                        Row(
                            modifier = Modifier.fillMaxWidth(),
                            horizontalArrangement =
                                Arrangement.SpaceBetween,
                            verticalAlignment = Alignment.CenterVertically
                        ) {
                            WhiteText(
                                "${monthNames[displayMonth]} $displayYear г.",
                                18,
                                true
                            )

                            Row {
                                SmallAction("‹") {
                                    if (displayMonth == 0) {
                                        displayMonth = 11
                                        displayYear--
                                    } else {
                                        displayMonth--
                                    }
                                }
                                Spacer(Modifier.width(8.dp))
                                SmallAction("›") {
                                    if (displayMonth == 11) {
                                        displayMonth = 0
                                        displayYear++
                                    } else {
                                        displayMonth++
                                    }
                                }
                            }
                        }

                        val weekdays = listOf(
                            "Пн", "Вт", "Ср", "Чт", "Пт", "Сб", "Вс"
                        )

                        Row(Modifier.fillMaxWidth()) {
                            weekdays.forEach {
                                Box(
                                    Modifier.weight(1f),
                                    contentAlignment = Alignment.Center
                                ) {
                                    GrayText(it)
                                }
                            }
                        }

                        val cal = Calendar.getInstance().apply {
                            clear()
                            set(displayYear, displayMonth, 1)
                        }

                        val offset =
                            (cal.get(Calendar.DAY_OF_WEEK) + 5) % 7
                        val total = cal.getActualMaximum(
                            Calendar.DAY_OF_MONTH
                        )
                        val cells = offset + total
                        val rows = (cells + 6) / 7

                        repeat(rows) { row ->
                            Row(
                                horizontalArrangement =
                                    Arrangement.spacedBy(5.dp)
                            ) {
                                repeat(7) { column ->
                                    val day = row * 7 + column - offset + 1

                                    if (day in 1..total) {
                                        val hasCare = records.any {
                                            it.day == day &&
                                                    it.month == displayMonth &&
                                                    it.year == displayYear
                                        }

                                        val selected =
                                            day == selectedDay &&
                                                    displayMonth == selectedMonth &&
                                                    displayYear == selectedYear

                                        Box(
                                            modifier = Modifier
                                                .weight(1f)
                                                .height(62.dp)
                                                .background(
                                                    when {
                                                        selected -> Color.White
                                                        hasCare -> Color(0xFF244737)
                                                        else -> Panel
                                                    },
                                                    RoundedCornerShape(9.dp)
                                                )
                                                .clickable {
                                                    selectedDay = day
                                                    selectedMonth = displayMonth
                                                    selectedYear = displayYear
                                                },
                                            contentAlignment = Alignment.Center
                                        ) {
                                            Text(
                                                "$day",
                                                color = if (selected)
                                                    Color.Black
                                                else Color.White,
                                                fontSize = 12.sp
                                            )
                                        }
                                    } else {
                                        Spacer(
                                            Modifier
                                                .weight(1f)
                                                .height(62.dp)
                                        )
                                    }
                                }
                            }
                            Spacer(Modifier.height(5.dp))
                        }

                        WhiteText(
                            "Записи на $selectedDay." +
                                    "${selectedMonth + 1}.$selectedYear",
                            15,
                            true
                        )

                        val dayRecords = records.filter {
                            it.day == selectedDay &&
                                    it.month == selectedMonth &&
                                    it.year == selectedYear
                        }

                        DarkCard {
                            if (dayRecords.isEmpty()) {
                                GrayText(
                                    "На эту дату записей ухода нет."
                                )
                            } else {
                                dayRecords.forEach { record ->
                                    val item = clothes.find {
                                        it.id == record.clothingId
                                    }
                                    WhiteText(
                                        "${item?.name ?: "Вещь"}: " +
                                                record.procedure
                                    )
                                }
                            }
                        }
                    }

                    "Статистика" -> {
                        SectionTitle(
                            "АНАЛИТИКА ГАРДЕРОБА",
                            "Статистика и износ"
                        )

                        val average = if (clothes.isEmpty()) 0
                        else (
                                clothes.map { it.condition }.average() * 100
                                ).toInt()

                        DarkCard {
                            Text(
                                "ОБЩИЙ ИНДЕКС СОСТОЯНИЯ",
                                color = Cyan,
                                fontSize = 11.sp
                            )
                            Text(
                                "$average%",
                                color = Green,
                                fontSize = 40.sp,
                                fontWeight = FontWeight.Bold
                            )
                            GrayText(
                                "Среднее состояние вещей в твоей коллекции."
                            )
                        }

                        Row(
                            horizontalArrangement =
                                Arrangement.spacedBy(8.dp)
                        ) {
                            StatBox(
                                clothes.size.toString(),
                                "Всего вещей",
                                Modifier.weight(1f)
                            )
                            StatBox(
                                records.size.toString(),
                                "Уходов записано",
                                Modifier.weight(1f)
                            )
                            StatBox(
                                clothes.count { it.needsCare }.toString(),
                                "Требуют внимания",
                                Modifier.weight(1f)
                            )
                        }

                        WhiteText("Состав гардероба", 16, true)

                        listOf(
                            "Обувь", "Одежда", "Аксессуары"
                        ).forEach { value ->
                            val count = clothes.count {
                                it.category == value
                            }
                            DarkCard {
                                WhiteText("$value · $count", 13, true)
                                LinearProgressIndicator(
                                    progress = {
                                        if (clothes.isEmpty()) 0f
                                        else count.toFloat() / clothes.size
                                    },
                                    modifier = Modifier.fillMaxWidth(),
                                    color = Cyan,
                                    trackColor = Color.DarkGray
                                )
                            }
                        }

                        WhiteText("Активность ухода", 16, true)
                        DarkCard {
                            GrayText(
                                "Всего выполненных процедур: ${records.size}"
                            )
                        }
                    }

                    "Справочник" -> {
                        SectionTitle(
                            "ПОЛЕЗНЫЕ ЗНАНИЯ",
                            "Справочник материалов"
                        )

                        GrayText(
                            "Подбирай уход в зависимости от материала вещи."
                        )

                        val guides = listOf(
                            Triple(
                                "Кожа",
                                "Мягкая щётка и крем",
                                "Удаляй пыль сухой тканью. " +
                                        "Используй подходящий крем."
                            ),
                            Triple(
                                "Замша и нубук",
                                "Щётка для замши",
                                "Чисти после высыхания. " +
                                        "Используй специальные средства."
                            ),
                            Triple(
                                "Мембрана (Gore-Tex)",
                                "Деликатная очистка",
                                "Следуй ярлыку производителя."
                            ),
                            Triple(
                                "Текстиль и сетка",
                                "Нейтральное средство",
                                "Удаляй загрязнения мягкой щёткой."
                            ),
                            Triple(
                                "Шерсть",
                                "Деликатный уход",
                                "Соблюдай температурный режим на ярлыке."
                            ),
                            Triple(
                                "Лакированная кожа",
                                "Мягкая салфетка",
                                "Избегай абразивов и сильного нагрева."
                            )
                        )

                        guides.forEach { guide ->
                            DarkCard {
                                WhiteText(guide.first, 15, true)
                                WhiteText(guide.second, 13)
                                GrayText(guide.third)
                            }
                        }

                        WhiteText("Общие правила ухода", 16, true)

                        DarkCard {
                            GrayText(
                                "• Проверяй рекомендации на ярлыке."
                            )
                            GrayText(
                                "• Тестируй средство на незаметном участке."
                            )
                            GrayText(
                                "• Высушивай вещи перед хранением."
                            )
                        }
                    }

                    "Настройки" -> {
                        SectionTitle("ПЕРСОНАЛИЗАЦИЯ", "Настройки")

                        DarkCard {
                            WhiteText("Мой гардероб", 17, true)
                            GrayText("Личный профиль")
                        }

                        SettingSwitch(
                            "Напоминания об уходе",
                            reminders
                        ) { reminders = it }

                        SettingSwitch(
                            "Советы по уходу",
                            tips
                        ) { tips = it }

                        SettingSwitch(
                            "Новости и обновления",
                            news
                        ) { news = it }

                        WhiteText("Приложение", 17, true)

                        DarkCard {
                            WhiteText("Справочник материалов")
                            GrayText("Помощь и FAQ")
                            GrayText("О приложении · версия 1.0")
                        }

                        GrayText(
                            "Настройки пока действуют только " +
                                    "в текущем запуске приложения."
                        )
                    }
                }
            }
        }
    }
}

@Composable
fun AppHeader(onSettings: () -> Unit) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .background(Panel)
            .padding(16.dp),
        verticalAlignment = Alignment.CenterVertically
    ) {
        Text("✦", color = Cyan, fontSize = 26.sp)
        Spacer(Modifier.width(12.dp))

        Column(Modifier.weight(1f)) {
            WhiteText("ГАРДЕРОБ МАСТЕР", 14, true)
            GrayText("Трекер чистки и ухода")
        }

        SmallAction("⚙", onSettings)
    }
}

@Composable
fun SectionTitle(kicker: String, title: String) {
    Text(
        kicker,
        color = Cyan,
        fontSize = 11.sp,
        fontWeight = FontWeight.Bold
    )
    WhiteText(title, 25, true)
}

@Composable
fun WhiteText(
    text: String,
    size: Int = 14,
    bold: Boolean = false
) {
    Text(
        text = text,
        color = Color.White,
        fontSize = size.sp,
        fontWeight = if (bold) FontWeight.Bold
        else FontWeight.Normal,
        maxLines = 3,
        overflow = TextOverflow.Ellipsis
    )
}

@Composable
fun GrayText(text: String) {
    Text(
        text,
        color = Muted,
        fontSize = 12.sp
    )
}

@Composable
fun DarkCard(
    content: @Composable ColumnScope.() -> Unit
) {
    Column(
        modifier = Modifier
            .fillMaxWidth()
            .background(Panel, RoundedCornerShape(14.dp))
            .padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp),
        content = content
    )
}

@Composable
fun StatBox(
    value: String,
    label: String,
    modifier: Modifier = Modifier
) {
    Column(
        modifier = modifier
            .background(Panel, RoundedCornerShape(12.dp))
            .padding(10.dp)
    ) {
        WhiteText(value, 23, true)
        GrayText(label)
    }
}

@Composable
fun ClothingCard(
    item: Clothing,
    onDetails: () -> Unit,
    onCare: () -> Unit
) {
    Column(
        modifier = Modifier
            .fillMaxWidth()
            .background(Panel, RoundedCornerShape(14.dp))
    ) {
        Box(
            modifier = Modifier
                .fillMaxWidth()
                .height(125.dp)
                .background(Color(0xFF292929)),
            contentAlignment = Alignment.Center
        ) {
            Text(item.symbol, fontSize = 44.sp)
            Text(
                if (item.needsCare) "УХОД" else "ЧИСТО",
                color = Green,
                fontSize = 10.sp,
                modifier = Modifier
                    .align(Alignment.TopStart)
                    .padding(10.dp)
            )
        }

        Column(
            modifier = Modifier.padding(8.dp),
            verticalArrangement = Arrangement.spacedBy(6.dp)
        ) {
            WhiteText(item.name, 12, true)
            GrayText("${item.category} · ${item.material}")

            LinearProgressIndicator(
                progress = { item.condition },
                modifier = Modifier.fillMaxWidth(),
                color = Pink,
                trackColor = Color.DarkGray
            )

            Row(
                horizontalArrangement = Arrangement.spacedBy(4.dp)
            ) {
                TinyButton(
                    "Подробнее",
                    Modifier.weight(1f),
                    onDetails
                )
                TinyButton(
                    "+ Уход",
                    Modifier.weight(1f),
                    onCare
                )
            }
        }
    }
}

@Composable
fun TinyButton(
    title: String,
    modifier: Modifier,
    onClick: () -> Unit
) {
    OutlinedButton(
        onClick = onClick,
        modifier = modifier,
        contentPadding = PaddingValues(2.dp),
        shape = RoundedCornerShape(8.dp)
    ) {
        Text(
            title,
            fontSize = 9.sp,
            color = Color.White
        )
    }
}


 <?xml version="1.0" encoding="utf-8"?>

<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="Гардероб Мастер"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.My_gorderob">


        <activity
            android:name="com.example.my_gorderob.MainActivity"
            android:exported="true">

            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>

        </activity>


    </application>

</manifest>
