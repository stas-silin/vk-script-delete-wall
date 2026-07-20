# vk-script-delete-wall
Данный скрипт удаляет все посты на стене во Вконтакте (проверено июль 2026)

СЛЕДУЙ ИНСТРУКЦИИ И ВСЁ ПОЛУЧИТСЯ!
1. Вам понадобится компьютер с браузером и небольшой фрагмент кода, который приведён ниже.
Время очистки будет зависеть от общего количества записей

3. Открывай группу Вконтакте - ты обязательно должен обладать правами Администратора

4. Открывай консоль браузера
Работаешь в Яндекс, Chrome, Опера - жми Ctrl + Shift + J (Windows) или Cmd + Opt + J (macOS). В Firefox для этого предусмотрены комбинации Ctrl + Shift + K (Windows) и Cmd + Opt + K (macOS).
Если используете другой браузер, клавиши для запуска его консоли вы легко найдёте с помощью поисковика.

5. Копируй код ниже и вставляй скрипт в консоль (пункт 4) - жми ENTER

6. Жди когда закончит. Скрипт автоматически прокручивает страницу вниз.
Ничего делать не надо - только ждать, и чтобы интернет не отваливался.

7. Поблагодари Вселенную, что наконец-то нашёл рабочий скрипт.

не получается?
Есть альтернативные скрипты здесь: https://ssilin.ru/blog/udalit-vse-posty-na-stene-vkontakte-2026-iyun/

Есть вопросы? Сюда: https://t.me/joytop

```javascript
    
const sleep = ms => new Promise(r => setTimeout(r, ms));

const STORAGE_KEY = 'vk_delete_progress_v1';

const CONFIG = {
  scrollStep: 900,
  scrollDelay: 120,
  pauseEveryScrolls: 50,
  preloadPause: 2000,

  deleteDelay: 450,
  maxFailsPerBatch: 30
};

function getProgress() {
  return JSON.parse(localStorage.getItem(STORAGE_KEY) || '{"totalDeleted":0,"batch":0}');
}

function saveProgress(data) {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
}

function resetProgress() {
  localStorage.removeItem(STORAGE_KEY);
  console.log('🧹 Прогресс очищен');
}

const sleepLog = async (msg, ms) => {
  console.log(msg);
  await sleep(ms);
};

async function waitFor(fn, timeout = 2500, interval = 100) {
  const start = Date.now();

  while (Date.now() - start < timeout) {
    const result = fn();
    if (result) return result;
    await sleep(interval);
  }

  return null;
}

function visible(el) {
  return el && el.offsetParent !== null;
}

function findByText(text) {
  return [...document.querySelectorAll('button, div[role="button"], span, div')]
    .find(el => visible(el) && el.textContent.trim() === text);
}

async function preloadDown(steps) {
  console.log(`⬇️ Прогружаю вниз: ${steps} шагов`);

  for (let i = 1; i <= steps; i++) {
    window.scrollBy(0, CONFIG.scrollStep);

    if (i % CONFIG.pauseEveryScrolls === 0) {
      console.log(`⏳ Прокручено ${i}/${steps}`);
      await sleep(CONFIG.preloadPause);
    } else {
      await sleep(CONFIG.scrollDelay);
    }
  }

  await sleepLog('⬆️ Возвращаюсь в начало', 500);

  window.scrollTo({
    top: 0,
    behavior: 'smooth'
  });

  await sleep(4000);
}

async function deleteOnePost() {
  const posts = [...document.querySelectorAll('[data-testid="post"]')];

  for (const post of posts) {
    const menuBtn = post.querySelector('[data-testid="post_context_menu_toggle"]');
    if (!menuBtn) continue;

    post.scrollIntoView({ block: 'center' });
    await sleep(220);

    menuBtn.click();
    await sleep(250);

    const deleteBtn = await waitFor(() => findByText('Удалить'), 2000);

    if (!deleteBtn) {
      document.body.click();
      await sleep(200);
      continue;
    }

    deleteBtn.click();
    await sleep(250);

    const confirmBtn = await waitFor(() => {
      return [...document.querySelectorAll('button, div[role="button"], span')]
        .find(el =>
          visible(el) &&
          el.textContent.trim() === 'Удалить' &&
          el !== deleteBtn
        );
    }, 1200);

    if (confirmBtn) {
      confirmBtn.click();
    }

    await sleep(CONFIG.deleteDelay);
    return true;
  }

  return false;
}

async function deleteBatch(limit) {
  let deletedInBatch = 0;
  let fails = 0;
  let progress = getProgress();

  console.log(`🧨 Удаляю пакет: ${limit} постов`);

  while (deletedInBatch < limit && fails < CONFIG.maxFailsPerBatch) {
    if (window.STOP_DELETE) {
      console.log('⛔ Остановлено вручную');
      return deletedInBatch;
    }

    const deleted = await deleteOnePost();

    if (deleted) {
      deletedInBatch++;
      progress.totalDeleted++;
      progress.batch++;

      saveProgress(progress);

      if (deletedInBatch % 25 === 0) {
        console.log(`📦 В пакете: ${deletedInBatch}/${limit} | Всего: ${progress.totalDeleted}`);
      }

      fails = 0;
    } else {
      fails++;

      window.scrollBy(0, window.innerHeight * 0.8);
      await sleep(900);
    }
  }

  console.log(`✅ Пакет завершён. Удалено в пакете: ${deletedInBatch}`);
  return deletedInBatch;
}

async function runVkDeleteBatches() {
  window.STOP_DELETE = false;

  const previous = getProgress();

  console.log(`💾 Сохранённый прогресс: всего удалено ${previous.totalDeleted}`);

  const steps = parseInt(
    prompt('Сколько шагов вниз прогружать за один цикл?', '300'),
    10
  );

  if (!steps || steps < 1) {
    console.log('❌ Отменено');
    return;
  }

  const batchLimit = parseInt(
    prompt('Сколько постов удалять за цикл?', String(steps)),
    10
  );

  if (!batchLimit || batchLimit < 1) {
    console.log('❌ Отменено');
    return;
  }

  while (!window.STOP_DELETE) {
    await preloadDown(steps);

    const deleted = await deleteBatch(batchLimit);

    if (deleted === 0) {
      console.log('⚠️ За цикл ничего не удалилось. Останавливаю.');
      break;
    }

    console.log('🔁 Повторяю цикл: прогрузка вниз → начало → удаление');
    await sleep(2500);
  }

  const finalProgress = getProgress();
  console.log(`🎉 Работа остановлена. Всего удалено: ${finalProgress.totalDeleted}`);
}

runVkDeleteBatches();

    
```    
