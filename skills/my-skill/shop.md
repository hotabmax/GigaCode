Если проект интернет магазин:
## Бизнес-логика корзины
1. Корзина — это СПИСОК ЖЕЛАНИЙ, НЕ РЕЗЕРВ. Товар НЕ вычитается со склада при добавлении в корзину.
2. Товар вычитается со склада ТОЛЬКО при оформлении заказа (checkout) — в транзакции с atomic().
3. При удалении товара из корзины — склад НЕ меняется. Просто удаляется CartItem.
4. При отмене заказа — ТОЛЬКО ТОГДА товар возвращается на склад.
5. При обновлении количества в корзине — проверяем stock_quantity >= quantity, но НЕ меняем склад.

## Шаблоны
1. НИКОГДА не используй фильтр |add для арифметики. Он конкатенирует строки.
   ❌ {{ stock_quantity|add:"-cart_in_cart" }}  → даст "10--0"
   ✅ Передавай рассчитанное значение из view: available_to_add = max(0, product.stock_quantity - cart_in_cart)
2. Все вычисления (суммы, количества, доступные остатки) — в view. В шаблоне только отображение.

## Сессии
1. Перед использованием request.session.session_key вызови request.session.save(), иначе может быть None.
2. В get_or_create_cart для анонимного пользователя:
   - request.session.modified = True
   - if not request.session.session_key: request.session.save()
   - cart = Cart.objects.create(session_key=request.session.session_key)

## Тестирование
1. Пиши интеграционные тесты для каждого сценария корзины:
   - add_to_cart → check stock unchanged → update quantity → remove → check stock unchanged → checkout → check stock decreased
   - cancel_order → check stock restored
2. Тестируй граничные случаи: quantity=0, quantity=-1, quantity > stock, пустая корзина, неавторизованный пользователь.
3. Проверяй, что stock_quantity НЕ меняется при операциях с корзиной (кроме checkout и cancel_order).

## Архитектура
1. Модель Order должна иметь property total_quantity = sum(items.quantity)
2. CartItem должен хранить price на момент покупки (не product.price, а item.price)
3. Все POST-формы должны иметь CSRF-токен и валидацию количества (min=1)