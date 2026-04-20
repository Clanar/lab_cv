Спочатку ми дослідили наданий датасет для розуміння структури даних, виявлення аномалій та визначення стратегії валідації. Перевірка на пропущені значення показала їх відсутність в обох вибірках:

<img width="323" height="215" alt="зображення" src="https://github.com/user-attachments/assets/95ad2a16-2665-4a7e-9907-883241bf6df8" />

<br><br>Крім того, зображень з обличчями більше в 4 рази, ніж зображень з монетами, але у навчальній і тестовій вибірках їх пропорції однакові:

<img width="1359" height="512" alt="Знімок екрана з 2026-04-16 21-48-09" src="https://github.com/user-attachments/assets/99455b67-c00e-468c-9ce8-e0bce0be5a33" />

<br><br>Переважна більшість фотографій містить невелику кількість об'єктів, в середньому 4 обличчя і 2 монети. Максимальна кількість облич становить 29, а монет 102:

<img width="1359" height="474" alt="Знімок екрана з 2026-04-16 21-50-48" src="https://github.com/user-attachments/assets/b9aa6fa4-a9fb-4456-bbd5-2f3f9ca71fdb" />

## Validation
Для того щоб локальна валідація корелювала з рейтингом лідерів, було проведено порівняльний аналіз декількох підходів до розділення датасету:

<br>Random Split та K-Fold Cross-Validation показали ідентичні результати, де проблемою було те, що середнє значення монет сильно відрізняється у розподілах:
<table border="0">
  <tr>
    <td><img width="400" src="https://github.com/user-attachments/assets/a07e4eaa-1db3-40cf-b8a3-4e35134272a6" /></td>
    <td><img width="400" src="https://github.com/user-attachments/assets/c16115fe-327b-4900-8b74-9c5d9b328341" /></td>
  </tr>
</table>

<br>Group-Based Split краще розподілив по середньому значенню, проте пропорції збились:
<img width="400" src="https://github.com/user-attachments/assets/e10aba01-10b5-4e11-ae45-56bf636f0f6c" />

<br>Adversarial Validation створив валідаційну вибірку, яка повністю складалась з облич і жодної монети:
<img width="400" src="https://github.com/user-attachments/assets/9d3e29d5-73db-4c20-964d-34e67833ce97" />

<br>Stratified Split зберіг майже ідеальні пропорції 80/20 і інші значення розподілились при цьому рівномірніше:<br>
<img width="400" src="https://github.com/user-attachments/assets/d66e1fc4-57c4-4385-91e2-cd6317a7ce61" />

## Classical Computer Vision

Для обрання алгоритму для виявлення облич було протестовано Haar Cascades, Local Binary Patterns (LBP) i Histogram of Oriented Gradients (HOG). Для виявлення монет - Hought Circle Transform, Watershed Algorithm i Contour Detection:
<br><br>Haar показав високу швидкість, але була велика кількість False Positives. Hought Circle Transform через відсутню гнучкість до розміру монет виявився дуже ненадійний:

<img width="1089" height="326" src="https://github.com/user-attachments/assets/d6eb0181-ccb0-45ae-9013-ad3751ba19ec" />

<br>HOG значно краще справився, оскільки виокремлює обличчя під різними нахилами і перекриттями і виявився найкращим з протестованих. Watershed Algorithm дуже сильно помилявся:

<img width="1575" height="281" src="https://github.com/user-attachments/assets/5888eda2-e99d-43df-9a2b-c4a5dc4226c0" />

<br>LBP справився швидше за Haar, проте помилок все ще дуже багато. Contour Detection найкраще справився з виявленням монет:

<img width="1318" height="296" src="https://github.com/user-attachments/assets/8fcaaf1b-6e87-4966-8cdb-5c5f1be8cad1" />

<br>Після тестування було прийняте рішення обрати Histogram of Oriented Gradients для облич і Contour Detection для монет. Після цього подали прогнози в Kaggle і отримали наступні результати:
<img width="818" height="85" src="https://github.com/user-attachments/assets/a2648472-d01d-4c3f-bc30-269667edb525" />

## Classical Computer Vision + Classical Machine Learning
Для кожного зображення з датасету було вилучено 10 ключових ознак:
- brightness
- contrast
- faces_haar
- faces_lbp
- faces_hog
- coins_hough
- coins_watershed
- coins_contours
- total_contour_area
- type_encoded

Після цього було протестовано різні моделі: Random Forest, XGBoost,RIdge Regression, Lasso Regression, Support Vector Regression. Кожна навчалась на всіх десятьох ознаках, а після відбору ознак залишались тільки значущі.

Random Forest показав, що coins_contours, coins_watershed та brightness дають моделі найбільше інформації, тоді як faces_haar, faces_lbp та type_encoded виявились незначущими. Після їх відбору точність трохи впала:

<table border="0">
  <tr>
    <td><img width="802" height="884" alt="зображення" src="https://github.com/user-attachments/assets/294f84ff-1ea9-4deb-a09c-52cbdbf72242" /></td>
    <td><img width="802" height="884" alt="зображення" src="https://github.com/user-attachments/assets/46d97a9e-6a20-4aca-8e27-3c198d4ee8df" /></td>
  </tr>
</table>

XGBoost віддав найбільшу вагу type_encoded, він одразу розділяв логіку дерева на два класи. Серед усіх моделей ця виявилась найрезультативнішою:
<table border="0">
  <tr>
    <td><img width="878" height="884" alt="зображення" src="https://github.com/user-attachments/assets/9a6e227a-31d2-45dd-952a-19b0b2eb429c" /></td>
    <td><img width="878" height="884" alt="зображення" src="https://github.com/user-attachments/assets/33857ad0-8e63-431a-9f5c-4bbbb4350b35" /></td>
  </tr>
</table>

Ridge Regression ефективно розподілив ваги між сильно скорельованими ознаками. Проте більш результативним для даної задачі виявився метод Lasso Regression:
<table border="0">
  <tr>
    <td><img width="1048" height="960" alt="зображення" src="https://github.com/user-attachments/assets/82d384c0-6f49-45dd-a13e-017c7381f422" /></td>
    <td><img width="1048" height="960" alt="зображення" src="https://github.com/user-attachments/assets/8b3103aa-b69c-4f0d-9033-44278622f60d" /></td>
  </tr>
</table>

Lasso Regression здійснив автоматичний відбір ознак, лінійно зменшивши коефіцієнти найменш інформативних параметрів до нуля:
<table border="0">
  <tr>
    <td><img width="1098" height="975" alt="зображення" src="https://github.com/user-attachments/assets/c50ce2e1-1476-419b-a492-eb6bd1c125b8" /></td>
    <td><img width="1098" height="975" alt="зображення" src="https://github.com/user-attachments/assets/6bc9fa88-6561-4832-a08d-07bdea32d794" /></td>
  </tr>
</table>

Support Vector Regression після видалення шумових ознак зміг обійти Lasso та Ridge Regression:
<table border="0">
  <tr>
    <td><img width="1098" height="975" alt="зображення" src="https://github.com/user-attachments/assets/cb5debf4-f7e2-428b-ad91-396dff50ef96" /></td>
    <td><img width="1098" height="975" alt="зображення" src="https://github.com/user-attachments/assets/1b64dd3d-39bb-4612-a372-ad9df0f90b95" /></td>
  </tr>
</table>

<br><br>Отже, найкращим рішенням виявився XGBoost з усіма ознаками, трохи обігнавши з вирізаними ознаками:
<img width="1077" height="168" alt="зображення" src="https://github.com/user-attachments/assets/60b74441-42b3-44a4-95c8-17138d9bb82b" />
