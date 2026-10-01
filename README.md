# ForelCards — Supabase + Storage

Ця версія оптимізована для 1000+ карток: зображення зберігаються в Supabase Storage, у таблиці `cards` зберігаються лише метадані та шлях до файлу. Каталог завантажує метадані без Base64-зображень і показує картки сторінками по 30.

## 1. Supabase SQL

Виконай SQL нижче в Supabase SQL Editor:

```sql
alter table public.cards add column if not exists image_path text;

-- Якщо таблиця cards вже існує, старе поле image можна залишити для одноразової міграції.
-- Нові записи більше не повинні записувати Base64 у image.

insert into storage.buckets (id, name, public)
values ('card-images', 'card-images', true)
on conflict (id) do update set public = true;

create policy "Public read card images"
on storage.objects for select
to public
using (bucket_id = 'card-images');

create policy "Public upload card images"
on storage.objects for insert
to public
with check (bucket_id = 'card-images');

create policy "Public update card images"
on storage.objects for update
to public
using (bucket_id = 'card-images')
with check (bucket_id = 'card-images');

create policy "Public delete card images"
on storage.objects for delete
to public
using (bucket_id = 'card-images');
```

> Для production краще обмежити insert/update/delete через Supabase Auth. Поточна адмінка вже історично працювала з клієнтським паролем, тому політики тут сумісні з цією схемою.

## 2. Конфіг

У `supabase-config.js` мають бути URL та publishable key вашого Supabase-проєкту.

## 3. Що виправлено

- Base64-картинки більше не завантажуються разом зі списком карток.
- Нові WebP завантажуються в Supabase Storage.
- Таблиця `cards` отримує тільки `image_path`.
- Каталог завантажує легкі метадані.
- Пагінація: 30 карток на сторінку.
- Lazy loading зображень.
- Пошук, фільтри та сортування працюють по всьому каталогу.
- Старі записи з Base64 підтримуються та завантажуються лише коли потрібна конкретна картинка.
- Адмін-список також не тягне всі великі зображення одразу.
