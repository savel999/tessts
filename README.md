# tessts


# Тесты не нужны
# Енамные типы

```
package types

type GenderType string

const (
	GenderTypeMale      GenderType = "male"
	GenderTypeFemale    GenderType = "female"
	GenderTypeUndefined GenderType = ""
)

func (t GenderType) IsValid() bool {
	switch t {
	case GenderTypeMale,
		GenderTypeFemale,
		GenderTypeUndefined:
		return true
	default:
		return false
	}
}
```


# Лучше протестировать только перемап
```
func GenderTypeForGQL(t types.GenderType) gen.Gender {
	switch t {
	case types.GenderTypeFemale:
		return gen.GenderFemale
	case types.GenderTypeMale:
		return gen.GenderMale
	default:
		return gen.GenderUndefined
	}
}
```




# Тесты для конфигов
# Минимализм в конфигах, чтобы проект собирался с минимальным набором энв-переменных(не забываем прокидывать энвы в make env и кубовые манифесты для стейджинга)

```
func initPostgresConfig(serviceName string) (*PostgresConfig, error) {
	maxOpenConnections, err := strconv.Atoi(os.Getenv("POSTGRES_MAX_OPEN_CONNECTIONS"))
	if err != nil {
		return nil, tracerr.Errorf("failed to get POSTGRES_MAX_OPEN_CONNECTIONS: %w", err)
	}

	maxIdleConnections, err := strconv.Atoi(os.Getenv("POSTGRES_MAX_IDLE_CONNECTIONS"))
	if err != nil {
		return nil, tracerr.Errorf("failed to get POSTGRES_MAX_IDLE_CONNECTIONS: %w", err)
	}

	readPostgresTimeout, err := strconv.Atoi(os.Getenv("POSTGRES_READ_TIMEOUT"))
	if err != nil {
		return nil, tracerr.Errorf("failed to get POSTGRES_READ_TIMEOUT: %w", err)
	}

	writePostgresTimeout, err := strconv.Atoi(os.Getenv("POSTGRES_WRITE_TIMEOUT"))
	if err != nil {
		return nil, tracerr.Errorf("failed to get POSTGRES_WRITE_TIMEOUT: %w", err)
	}

	return &PostgresConfig{
		Dsn:                os.Getenv("POSTGRES_DSN"),
		DBPoolSize:         DefaultDBPoolSizeMultiplier * runtime.NumCPU(),
		ServiceName:        serviceName,
		MaxOpenConnections: maxOpenConnections,
		MaxIdleConnections: maxIdleConnections,
		ReadTimeout:        time.Duration(readPostgresTimeout) * time.Millisecond,
		WriteTimeout:       time.Duration(writePostgresTimeout) * time.Millisecond,
	}, nil
}
```
# Необязательные поля заменять на дефолтные

```
func initPostgresConfig(serviceName string) *PostgresConfig {
	return &PostgresConfig{
		Dsn:                os.Getenv("POSTGRES_DSN"),
		DBPoolSize:         DefaultDBPoolSizeMultiplier * runtime.NumCPU(),
		ServiceName:        serviceName,
		MaxOpenConnections: getEnvInt("POSTGRES_MAX_OPEN_CONNECTIONS", DefaultDBMaxOpenConnections),
		MaxIdleConnections: getEnvInt("POSTGRES_MAX_IDLE_CONNECTIONS", DefaultDBMaxIdleConnections),
		ReadTimeout:        getEnvDuration("POSTGRES_READ_TIMEOUT", DefaultDBReadTimeout),
		WriteTimeout:       getEnvDuration("POSTGRES_WRITE_TIMEOUT", DefaultDBWriteTimeout),
	}
}
```


# Тесты нужны

```
func switchQwertyLayout(raw string) string {
	result := ""
	qwertyMap := getQwertyMap()

	for _, letter := range raw {
		if val, ok := qwertyMap[letter]; ok {
			result += string(val)

			continue
		}

		result += string(letter)
	}

	return result
}

// EscapeTemplateSearchByWords - поиск, путём разбиения исходной строки на слова
// withQwerty - поиск по обратной раскладке
// Примеры:
// "Кунаки — в мини" -> ".*\mКунаки.*\mв.*\mмини"
// "пробовали: 6 рецептов" -> ".*\mпробовали\:.*\m6.*\mрецептов"
// "ghj,jdfkb^ 6 htwtgnjd" -> ".*\mпробовали\:.*\m6.*\mрецептов" (с режимом qwerty)
// "Total-black : Эмили Ратаковски" -> ".*\mTotal\-black.*\:.*\mЭмили.*\mРатаковски"
func EscapeTemplateSearchByWords(searchVal string, withQwerty bool) string {
	result := ""

	if withQwerty {
		result = sanitizeSearchString(switchQwertyLayout(searchVal))
	} else {
		result = sanitizeSearchString(searchVal)
	}

	return getSearchTemplateForPostgres(EscapeRegexpForPostgres(result))
}
```

# Тесты в go-packages
# Стремиться к 100% покрытию https://i.imgur.com/ARRc7QY.png -> https://i.imgur.com/9WbfDX0.png 
# Test_Folder(right click) More Run/Debug -> Run with coverage -> Go test #Test_Folder#) 

```
func calculateSize(origWidth, origHeight, width, height int) (int, int) {
	var w, h int

	ratio := float64(origWidth) / float64(origHeight)
	smallerWidth := origWidth <= width
	smallerHeight := origHeight <= height

	if ratio > (float64(width) / float64(height)) {
		if smallerWidth {
			w = origWidth
			h = origHeight
		} else {
			w = width
			h = int(math.Ceil(float64(width) / ratio))
		}

		return w, h
	}

	if smallerHeight {
		w = origWidth
		h = origHeight
	} else {
		w = int(math.Ceil(float64(height) * ratio))
		h = height
	}

	return w, h
}
```
