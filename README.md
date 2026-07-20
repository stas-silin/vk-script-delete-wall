# vk-script-delete-wall
Данный скрипт удаляет все посты на стене во Вконтакте


Что нужно, чтобы удалить все посты / записи на стене группы Вконтакте
(рабочий способ - проверено в июне 2026)
1. Вам понадобится компьютер с браузером и небольшой фрагмент кода, который приведён ниже.
Время очистки будет зависеть от общего количества записей

3. Открывай группу Вконтакте - ты обязательно должен обладать правами Администратора

4. Открывай консоль браузера
Работаешь в Яндекс, Chrome, Опера - жми Ctrl + Shift + J (Windows) или Cmd + Opt + J (macOS). В Firefox для этого предусмотрены комбинации Ctrl + Shift + K (Windows) и Cmd + Opt + K (macOS).
Если используете другой браузер, клавиши для запуска его консоли вы легко найдёте с помощью Google.

5. Копируй и вставляй скрипт - жми ENTER

6. Жди когда закончит. Скрипт автоматически прокручивает страницу вниз.
Ничего делать не надо - только ждать, и чтобы интернет не отваливался.

7. Поблагодари Вселенную, что наконец-то нашёл рабочий скрипт.


async function deleteAllPosts() {
    function findMenuButton(post) {
        return post.querySelector('[data-testid="post_context_menu_toggle"]');
    }
    
    function findDeleteButton() {
        let allElements = document.querySelectorAll('*');
        for (let el of allElements) {
            if (el.textContent && el.textContent.trim() === 'Удалить') {
                if (el.offsetParent !== null) return el;
            }
        }
        return null;
    }
    
    let lastCount = -1;
    let emptyScrolls = 0;
    
    while (true) {
        let posts = document.querySelectorAll('[data-testid="post"]');
        if (posts.length === 0) {
            console.log('❌ Посты не найдены');
            break;
        }
        console.log(`📦 Найдено постов: ${posts.length}`);
        let deleted = 0;
        
        for (let i = 0; i < posts.length; i++) {
            let post = posts[i];
            let menuBtn = findMenuButton(post);
            if (!menuBtn) continue;
            
            post.scrollIntoView({ behavior: 'smooth', block: 'center' });
            await new Promise(r => setTimeout(r, 300));
            
            menuBtn.click();
            await new Promise(r => setTimeout(r, 500));
            
            let deleteBtn = findDeleteButton();
            if (deleteBtn) {
                deleteBtn.click();
                console.log(`✅ Пост ${i+1} удалён`);
                deleted++;
                await new Promise(r => setTimeout(r, 800));
                
                let confirmBtn = document.querySelector('button[type="submit"]');
                if (confirmBtn && confirmBtn.innerText.includes('Удалить')) {
                    confirmBtn.click();
                    await new Promise(r => setTimeout(r, 300));
                }
            } else {
                console.log(`⚠️ Пост ${i+1}: пункт "Удалить" не найден`);
                document.body.click();
            }
            await new Promise(r => setTimeout(r, 500));
        }
        
        if (deleted === 0) {
            emptyScrolls++;
            if (emptyScrolls > 2) break;
        } else {
            emptyScrolls = 0;
        }
        
        window.scrollBy(0, window.innerHeight);
        await new Promise(r => setTimeout(r, 2000));
        
        let newPosts = document.querySelectorAll('[data-testid="post"]');
        if (newPosts.length === lastCount && deleted === 0) break;
        lastCount = newPosts.length;
    }
    console.log('🎉 Удаление завершено!');
}

deleteAllPosts();
