# *Даталогическая модель для PostgreSQL*

## Таблица Role

| Поле | Тип | Ограничения | Описание |
|------|-----|-------------|----------|
| role_id | SERIAL | PRIMARY KEY | идентификатор роли |
| role_name | VARCHAR(50) | NOT NULL, UNIQUE | название роли (ADMIN, CLIENT и др.) |
| role_description | VARCHAR(250) | NULLABLE | описание полномочий роли |

**Индексы:**
- `idx_role_name` ON (role_name)

---

## Таблица User

| Поле | Тип | Ограничения | Описание |
|------|-----|-------------|----------|
| usr_id | SERIAL | PRIMARY KEY | идентификатор пользователя |
| usr_login | VARCHAR(50) | NOT NULL, UNIQUE | имя аккаунта пользователя |
| usr_email | VARCHAR(100) | NOT NULL, UNIQUE, CHECK (usr_email ~* '^[^@]+@[^@]+\.[^@]+$') | адрес электронной почты |
| usr_password | VARCHAR(255) | NOT NULL | хэш пароля пользователя |
| usr_is_active | BOOLEAN | NOT NULL, DEFAULT TRUE | статус активности аккаунта |
| usr_created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | время создания аккаунта |

**Индексы:**
- `idx_user_login` ON (usr_login)
- `idx_user_email` ON (usr_email)
- `idx_user_is_active` ON (usr_is_active)

---

## Таблица UserRole (M:N между User и Role)

| Поле | Тип | Ограничения | Описание |
|------|-----|-------------|----------|
| ur_id | SERIAL | PRIMARY KEY | идентификатор записи |
| ur_usr_id | INT | NOT NULL, FOREIGN KEY → User(usr_id) ON DELETE CASCADE | пользователь |
| ur_role_id | INT | NOT NULL, FOREIGN KEY → Role(role_id) ON DELETE RESTRICT | роль |
| ur_assigned_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | время назначения роли |

**Уникальность:**
- `UNIQUE (ur_usr_id, ur_role_id)` — нельзя назначить одну роль дважды

**Индексы:**
- `idx_userrole_usr_id` ON (ur_usr_id)
- `idx_userrole_role_id` ON (ur_role_id)

---

## Таблица UserProfile

| Поле | Тип | Ограничения | Описание |
|------|-----|-------------|----------|
| prf_usr_id | INT | PRIMARY KEY, FOREIGN KEY → User(usr_id) ON DELETE CASCADE | идентификатор пользователя |
| prf_display_name | VARCHAR(100) | NOT NULL | отображаемое имя |
| prf_phone | VARCHAR(20) | NULLABLE | контактный телефон |
| prf_avatar_url | VARCHAR(255) | NULLABLE | ссылка на аватар |
| prf_updated_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | время последнего обновления профиля |

---

## Таблица Permission

| Поле | Тип | Ограничения | Описание |
|------|-----|-------------|----------|
| perm_id | SERIAL | PRIMARY KEY | идентификатор права |
| perm_name | VARCHAR(100) | NOT NULL, UNIQUE | название права (CREATE_ORDER и др.) |
| perm_description | VARCHAR(250) | NULLABLE | описание права доступа |

**Индексы:**
- `idx_permission_name` ON (perm_name)

---

## Таблица RolePermission (M:N между Role и Permission)

| Поле | Тип | Ограничения | Описание |
|------|-----|-------------|----------|
| rp_id | SERIAL | PRIMARY KEY | идентификатор записи |
| rp_role_id | INT | NOT NULL, FOREIGN KEY → Role(role_id) ON DELETE CASCADE | роль |
| rp_perm_id | INT | NOT NULL, FOREIGN KEY → Permission(perm_id) ON DELETE CASCADE | право |

**Уникальность:**
- `UNIQUE (rp_role_id, rp_perm_id)` — нельзя назначить одно право одной роли дважды

**Индексы:**
- `idx_rolepermission_role_id` ON (rp_role_id)
- `idx_rolepermission_perm_id` ON (rp_perm_id)

---

## Таблица ActionLog

| Поле | Тип | Ограничения | Описание |
|------|-----|-------------|----------|
| log_id | BIGSERIAL | PRIMARY KEY | идентификатор записи журнала |
| log_usr_id | INT | NULLABLE, FOREIGN KEY → User(usr_id) ON DELETE SET NULL | пользователь, совершивший действие |
| log_action | VARCHAR(100) | NOT NULL | название выполненного действия |
| log_entity | VARCHAR(50) | NOT NULL | объект действия (ORDER, USER и др.) |
| log_ip_address | VARCHAR(45) | NULLABLE | IP-адрес клиента (IPv4/IPv6) |
| log_description | VARCHAR(500) | NULLABLE | полное описание действия |
| log_result | VARCHAR(20) | NOT NULL, CHECK (log_result IN ('SUCCESS', 'FAIL')) | результат |
| log_timestamp | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | время создания записи |

**Индексы:**
- `idx_actionlog_usr_id` ON (log_usr_id)
- `idx_actionlog_timestamp` ON (log_timestamp)
- `idx_actionlog_entity` ON (log_entity)

---

## Таблица Address

| Поле | Тип | Ограничения | Описание |
|------|-----|-------------|----------|
| addr_id | SERIAL | PRIMARY KEY | идентификатор адреса |
| addr_usr_id | INT | NOT NULL, FOREIGN KEY → User(usr_id) ON DELETE CASCADE | владелец адреса |
| addr_city | VARCHAR(100) | NOT NULL | город |
| addr_street | VARCHAR(150) | NOT NULL | улица |
| addr_building | VARCHAR(20) | NOT NULL | дом |
| addr_apartment | VARCHAR(20) | NULLABLE | квартира |
| addr_comment | VARCHAR(250) | NULLABLE | комментарий к адресу |

**Индексы:**
- `idx_address_usr_id` ON (addr_usr_id)
- `idx_address_city` ON (addr_city)

---

## Таблица Category

| Поле | Тип | Ограничения | Описание |
|------|-----|-------------|----------|
| cat_id | SERIAL | PRIMARY KEY | идентификатор категории |
| cat_name | VARCHAR(100) | NOT NULL, UNIQUE | название категории (Пицца, Напитки) |
| cat_description | VARCHAR(250) | NULLABLE | описание категории |
| cat_is_active | BOOLEAN | NOT NULL, DEFAULT TRUE | статус активности категории |

**Индексы:**
- `idx_category_name` ON (cat_name)
- `idx_category_is_active` ON (cat_is_active)

---

## Таблица Product

| Поле | Тип | Ограничения | Описание |
|------|-----|-------------|----------|
| prod_id | SERIAL | PRIMARY KEY | идентификатор товара |
| prod_cat_id | INT | NOT NULL, FOREIGN KEY → Category(cat_id) ON DELETE RESTRICT | категория товара |
| prod_name | VARCHAR(150) | NOT NULL | название товара |
| prod_description | VARCHAR(500) | NULLABLE | описание товара |
| prod_photo_url | VARCHAR(255) | NULLABLE | ссылка на фото товара |
| prod_composition | VARCHAR(500) | NULLABLE | состав блюда |
| prod_price | NUMERIC(10,2) | NOT NULL, CHECK (prod_price >= 0) | базовая цена товара |
| prod_is_active | BOOLEAN | NOT NULL, DEFAULT TRUE | статус активности товара |

**Индексы:**
- `idx_product_cat_id` ON (prod_cat_id)
- `idx_product_name` ON (prod_name)
- `idx_product_is_active` ON (prod_is_active)

---

## Таблица ProductSize

| Поле | Тип | Ограничения | Описание |
|------|-----|-------------|----------|
| size_id | SERIAL | PRIMARY KEY | идентификатор размера |
| size_prod_id | INT | NOT NULL, FOREIGN KEY → Product(prod_id) ON DELETE CASCADE | товар |
| size_name | VARCHAR(10) | NOT NULL | название размера (S, M, L) |
| size_price | NUMERIC(10,2) | NOT NULL, CHECK (size_price >= 0) | цена для данного размера |

**Уникальность:**
- `UNIQUE (size_prod_id, size_name)` — у одного товара не может быть двух одинаковых размеров

**Индексы:**
- `idx_productsize_prod_id` ON (size_prod_id)

---

## Таблица Order

| Поле | Тип | Ограничения | Описание |
|------|-----|-------------|----------|
| ord_id | BIGSERIAL | PRIMARY KEY | идентификатор заказа |
| ord_usr_id | INT | NOT NULL, FOREIGN KEY → User(usr_id) ON DELETE RESTRICT | клиент |
| ord_type | VARCHAR(20) | NOT NULL, CHECK (ord_type IN ('DELIVERY','PICKUP')) | способ получения |
| ord_addr_id | INT | NULLABLE, FOREIGN KEY → Address(addr_id) ON DELETE SET NULL | адрес доставки |
| ord_pickup_point | VARCHAR(150) | NULLABLE | точка самовывоза |
| ord_contact_phone | VARCHAR(20) | NOT NULL | контактный телефон |
| ord_comment | VARCHAR(500) | NULLABLE | комментарий к заказу |
| ord_total | NUMERIC(10,2) | NOT NULL, CHECK (ord_total >= 0) | итоговая сумма заказа |
| ord_status | VARCHAR(20) | NOT NULL, DEFAULT 'NEW', CHECK (ord_status IN ('NEW','COOKING','DELIVERING','DONE','CANCELLED')) | статус заказа |
| ord_created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | время создания заказа |

**CHECK-ограничение:**
- `(ord_type = 'DELIVERY' AND ord_addr_id IS NOT NULL) OR (ord_type = 'PICKUP' AND ord_pickup_point IS NOT NULL)`

**Индексы:**
- `idx_order_usr_id` ON (ord_usr_id)
- `idx_order_status` ON (ord_status)
- `idx_order_created_at` ON (ord_created_at)
- `idx_order_type` ON (ord_type)

---

## Таблица OrderItem

| Поле | Тип | Ограничения | Описание |
|------|-----|-------------|----------|
| oi_id | BIGSERIAL | PRIMARY KEY | идентификатор позиции |
| oi_ord_id | BIGINT | NOT NULL, FOREIGN KEY → Order(ord_id) ON DELETE CASCADE | заказ |
| oi_prod_id | INT | NOT NULL, FOREIGN KEY → Product(prod_id) ON DELETE RESTRICT | товар |
| oi_size_id | INT | NULLABLE, FOREIGN KEY → ProductSize(size_id) ON DELETE SET NULL | размер товара |
| oi_quantity | INT | NOT NULL, CHECK (oi_quantity > 0) | количество |
| oi_price | NUMERIC(10,2) | NOT NULL, CHECK (oi_price >= 0) | цена позиции на момент заказа |

**Индексы:**
- `idx_orderitem_ord_id` ON (oi_ord_id)
- `idx_orderitem_prod_id` ON (oi_prod_id)
- `idx_orderitem_size_id` ON (oi_size_id)

---

## Таблица Payment

| Поле | Тип | Ограничения | Описание |
|------|-----|-------------|----------|
| pay_id | BIGSERIAL | PRIMARY KEY | идентификатор платежа |
| pay_ord_id | BIGINT | NOT NULL, FOREIGN KEY → Order(ord_id) ON DELETE RESTRICT | заказ |
| pay_amount | NUMERIC(10,2) | NOT NULL, CHECK (pay_amount >= 0) | сумма платежа |
| pay_method | VARCHAR(50) | NOT NULL, CHECK (pay_method IN ('CARD','CASH','ONLINE')) | способ оплаты |
| pay_status | VARCHAR(20) | NOT NULL, DEFAULT 'PENDING', CHECK (pay_status IN ('PENDING','COMPLETED','FAILED','REFUNDED')) | статус платежа |
| pay_created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | время проведения платежа |

**Индексы:**
- `idx_payment_ord_id` ON (pay_ord_id)
- `idx_payment_status` ON (pay_status)
- `idx_payment_created_at` ON (pay_created_at)

---

## Таблица Promocode

| Поле | Тип | Ограничения | Описание |
|------|-----|-------------|----------|
| promo_id | SERIAL | PRIMARY KEY | идентификатор промокода |
| promo_code | VARCHAR(50) | NOT NULL, UNIQUE | код промокода |
| promo_discount | NUMERIC(10,2) | NOT NULL, CHECK (promo_discount >= 0) | размер скидки |
| promo_is_active | BOOLEAN | NOT NULL, DEFAULT TRUE | статус активности промокода |
| promo_expires_at | TIMESTAMP | NULLABLE | срок действия промокода |

**Индексы:**
- `idx_promocode_code` ON (promo_code)
- `idx_promocode_is_active` ON (promo_is_active)
- `idx_promocode_expires_at` ON (promo_expires_at)

---

## Таблица OrderPromocode (M:N между Order и Promocode)

| Поле | Тип | Ограничения | Описание |
|------|-----|-------------|----------|
| op_id | BIGSERIAL | PRIMARY KEY | идентификатор записи |
| op_ord_id | BIGINT | NOT NULL, FOREIGN KEY → Order(ord_id) ON DELETE CASCADE | заказ |
| op_promo_id | INT | NOT NULL, FOREIGN KEY → Promocode(promo_id) ON DELETE RESTRICT | промокод |
| op_applied_discount | NUMERIC(10,2) | NOT NULL, CHECK (op_applied_discount >= 0) | применённая скидка (снимок на момент заказа) |
| op_applied_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | когда применён промокод |

**Уникальность:**
- `UNIQUE (op_ord_id, op_promo_id)` — один промокод нельзя применить к заказу дважды

**Индексы:**
- `idx_orderpromocode_ord_id` ON (op_ord_id)
- `idx_orderpromocode_promo_id` ON (op_promo_id)

---

## Таблица OrderStatusHistory

| Поле | Тип | Ограничения | Описание |
|------|-----|-------------|----------|
| osh_id | BIGSERIAL | PRIMARY KEY | идентификатор записи |
| osh_ord_id | BIGINT | NOT NULL, FOREIGN KEY → Order(ord_id) ON DELETE CASCADE | заказ |
| osh_status | VARCHAR(20) | NOT NULL, CHECK (osh_status IN ('NEW','COOKING','DELIVERING','DONE','CANCELLED')) | статус заказа |
| osh_changed_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | время изменения статуса |
| osh_changed_by | INT | NULLABLE, FOREIGN KEY → User(usr_id) ON DELETE SET NULL | пользователь, изменивший статус |

**Индексы:**
- `idx_orderstatushistory_ord_id` ON (osh_ord_id)
- `idx_orderstatushistory_changed_at` ON (osh_changed_at)
- `idx_orderstatushistory_changed_by` ON (osh_changed_by)

---

## Таблица Delivery

| Поле | Тип | Ограничения | Описание |
|------|-----|-------------|----------|
| del_id | BIGSERIAL | PRIMARY KEY | идентификатор доставки |
| del_ord_id | BIGINT | NOT NULL, UNIQUE, FOREIGN KEY → Order(ord_id) ON DELETE CASCADE | заказ (1:1) |
| del_courier_id | INT | NULLABLE, FOREIGN KEY → User(usr_id) ON DELETE SET NULL | курьер |
| del_status | VARCHAR(20) | NOT NULL, DEFAULT 'ASSIGNED', CHECK (del_status IN ('ASSIGNED','IN_TRANSIT','DELIVERED','FAILED')) | статус доставки |
| del_delivered_at | TIMESTAMP | NULLABLE | время доставки |

**Уникальность:**
- `UNIQUE (del_ord_id)` — у одного заказа только одна доставка (связь 1:1)

**Индексы:**
- `idx_delivery_ord_id` ON (del_ord_id)
- `idx_delivery_courier_id` ON (del_courier_id)
- `idx_delivery_status` ON (del_status)

---

## Таблица связей

| Связь | Тип | Реализация | ON DELETE |
|-------|-----|------------|-----------|
| User ↔ Role | **M:N** | `UserRole` | CASCADE / RESTRICT |
| Role ↔ Permission | **M:N** | `RolePermission` | CASCADE / CASCADE |
| Order ↔ Promocode | **M:N** | `OrderPromocode` | CASCADE / RESTRICT |
| User → UserProfile | 1:1 | FK `prf_usr_id` | CASCADE |
| User → Address | 1:N | FK `addr_usr_id` | CASCADE |
| User → Order | 1:N | FK `ord_usr_id` | RESTRICT |
| User → ActionLog | 1:N | FK `log_usr_id` | SET NULL |
| User → OrderStatusHistory | 1:N | FK `osh_changed_by` | SET NULL |
| User → Delivery | 1:N | FK `del_courier_id` | SET NULL |
| Category → Product | 1:N | FK `prod_cat_id` | RESTRICT |
| Product → ProductSize | 1:N | FK `size_prod_id` | CASCADE |
| Product → OrderItem | 1:N | FK `oi_prod_id` | RESTRICT |
| ProductSize → OrderItem | 1:N | FK `oi_size_id` | SET NULL |
| Order → OrderItem | 1:N | FK `oi_ord_id` | CASCADE |
| Order → Payment | 1:N | FK `pay_ord_id` | RESTRICT |
| Order → OrderStatusHistory | 1:N | FK `osh_ord_id` | CASCADE |
| Order → Delivery | 1:1 | FK `del_ord_id` UNIQUE | CASCADE |
| Address → Order | 1:N | FK `ord_addr_id` | SET NULL |

---
