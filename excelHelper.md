# Excel Formula Reference

## 1. Merge Two Columns

### Recommended (Modern Excel)
```excel
=B2 & "/" & E2
```

### Older Alternative
```excel
=CONCATENATE(B2,"/",E2)
```

### Example

| B2 | E2 | Output |
|---|---|---|
| Sales | India | Sales/India |

### What it does
Combines values from two columns with `/` between them.

---

## 2. Add Prefix and Remove Quotes

```excel
=IF(TRIM(A1)="","", "zcrm_" & SUBSTITUTE(A1,"""",""))
```

### Example

| A1 Input | Output |
|---|---|
| Customer | zcrm_Customer |
| "Lead" | zcrm_Lead |
| *(blank)* | *(blank)* |

### What it does
- Checks if `A1` is empty
- Removes double quotes (`"`)
- Adds `zcrm_` prefix

---

## 3. Get Text After `/`

```excel
=IF(ISNUMBER(SEARCH("/",AA2)), TRIM(TEXTAFTER(AA2,"/")), "")
```

### Compatible Version (Older Excel)
```excel
=IF(ISNUMBER(SEARCH("/",AA2)), TRIM(MID(AA2, SEARCH("/",AA2)+1, LEN(AA2))), "")
```

### Example

| AA2 Input | Output |
|---|---|
| Sales/India | India |
| Dept / HR | HR |
| Finance | *(blank)* |

### What it does
Returns everything after `/`.

---

## 4. Get Text Before `/`

```excel
=IF(ISNUMBER(SEARCH("/",AA2)), TRIM(TEXTBEFORE(AA2,"/")), AA2)
```

### Compatible Version (Older Excel)
```excel
=IF(ISNUMBER(SEARCH("/",AA2)), LEFT(AA2, SEARCH("/",AA2)-1), AA2)
```

### Example

| AA2 Input | Output |
|---|---|
| Sales/India | Sales |
| Dept / HR | Dept |
| Finance | Finance |

### What it does
Returns everything before `/`.

---

## 5. Get Text Before `TO`

### Recommended
```excel
=IF(BR2="","",TRIM(TEXTBEFORE(BR2,"TO")))
```

### Compatible Version (Older Excel)
```excel
=IF(BR2="","",LEFT(BR2,FIND("TO",BR2)-2))
```

### Example

| BR2 Input | Output |
|---|---|
| DELHI TO MUMBAI | DELHI |
| SALES TO HR | SALES |

### What it does
Extracts text before the word `TO`.

---