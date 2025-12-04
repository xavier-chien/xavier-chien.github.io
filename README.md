theme: jekyll-theme-minimal
title: Octocat's homepage
description: Bookmark this to keep an eye on my project updates!
# xavier-chien.github.io
測試用
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>待辦清單 (Todo List) - Local Storage 版</title>
  
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://unpkg.com/react@18/umd/react.development.js"></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  

  <style>
    @import url('https://fonts.googleapis.com/css2?family=Inter:wght@100..900&display=swap');
    body {
      font-family: 'Inter', sans-serif;
    }
  </style>
</head>
<body class="bg-gray-100 min-h-screen">

<div id="root"></div>

<script type="text/babel">
  const { useState, useEffect, useCallback } = React;
  const { ListChecks, Plus, Trash2, CheckCircle, Circle } = window['lucide']; 
  const LOCAL_STORAGE_KEY = 'todoListAppTasks';
  
  const getTodosFromStorage = () => {
    try {
      const savedTasks = localStorage.getItem(LOCAL_STORAGE_KEY);
      return savedTasks ? JSON.parse(savedTasks) : [];
    } catch (error) {
      console.error("讀取 Local Storage 失敗:", error);
      return [];
    }
  };

  const saveTodosToStorage = (todos) => {
    try {
      localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify(todos));
      console.log(`待辦清單已儲存。總數: ${todos.length}`);
    } catch (error) {
      console.error("寫入 Local Storage 失敗:", error);
    }
  };

  const TodoItem = React.memo(({ todo, onToggle, onDelete }) => (
    <div className={`flex items-center p-3 sm:p-4 rounded-lg shadow-sm mb-3 transition duration-200 ease-in-out border 
                    ${todo.completed ? 'bg-green-50 border-green-200' : 'bg-white border-gray-200 hover:shadow-md'}`}>
      
      <button 
        onClick={() => onToggle(todo.id)}
        className="w-8 h-8 flex items-center justify-center text-gray-400 hover:text-indigo-600 transition duration-150 mr-3 flex-shrink-0"
        aria-label={todo.completed ? '標記為未完成' : '標記為完成'}
      >
        {todo.completed 
          ? <CheckCircle className="w-6 h-6 text-green-500 fill-green-500" /> 
          : <Circle className="w-6 h-6" />
        }
      </button>
      <span className={`flex-grow text-lg sm:text-xl font-medium break-words
                       ${todo.completed ? 'line-through text-gray-400' : 'text-gray-700'}`}>
        {todo.text}
      </span>

      <button 
        onClick={() => onDelete(todo.id)}
        className="ml-4 p-2 rounded-full text-red-400 hover:bg-red-100 hover:text-red-600 transition duration-150 flex-shrink-0"
        aria-label="刪除任務"
      >
        <Trash2 className="w-5 h-5" />
      </button>
    </div>
  ));

  const App = () => {
    const [todos, setTodos] = useState(getTodosFromStorage);
    const [newTodoText, setNewTodoText] = useState('');

    useEffect(() => {
      saveTodosToStorage(todos);
    }, [todos]); 


    const handleAddTodo = useCallback((e) => {
      e.preventDefault();
      const trimmedText = newTodoText.trim();
      if (!trimmedText) return;

      const newTodo = {

        id: Date.now(), 
        text: trimmedText,
        completed: false,
      };

      setTodos(prevTodos => [newTodo, ...prevTodos]); // 新的放前面
      setNewTodoText(''); // 清空輸入欄
    }, [newTodoText]);

    const handleToggleTodo = useCallback((id) => {
      setTodos(prevTodos => 
        prevTodos.map(todo => 
          todo.id === id ? { ...todo, completed: !todo.completed } : todo
        )
      );
    }, []);

    const handleDeleteTodo = useCallback((id) => {
      setTodos(prevTodos => prevTodos.filter(todo => todo.id !== id));
    }, []);

    const totalTasks = todos.length;
    const completedTasks = todos.filter(t => t.completed).length;

    return (
      <div className="min-h-screen flex flex-col items-center justify-start py-8 px-4">
        <div className="w-full max-w-2xl bg-white shadow-2xl rounded-2xl p-6 md:p-10 border-t-8 border-indigo-600">
          
          {/* 標題與統計區 */}
          <h1 className="text-3xl sm:text-4xl font-extrabold text-center text-indigo-700 mb-2">
            <ListChecks className="inline w-8 h-8 mr-2 text-indigo-500" />
            個人待辦清單
          </h1>
          <p className="text-center text-gray-500 mb-6">
            您專屬的純前端任務管理器 (Local Storage 持久化)
          </p>
          
          {/* 統計面板 */}
          <div className="bg-indigo-50 p-4 rounded-lg mb-8 shadow-inner flex justify-around text-center">
            <div>
              <p className="text-2xl font-bold text-indigo-800">{totalTasks}</p>
              <p className="text-sm text-gray-600">總任務數</p>
            </div>
            <div>
              <p className="text-2xl font-bold text-green-600">{completedTasks}</p>
              <p className="text-sm text-gray-600">已完成</p>
            </div>
            <div>
              <p className="text-2xl font-bold text-red-600">{totalTasks - completedTasks}</p>
              <p className="text-sm text-gray-600">待辦中</p>
            </div>
          </div>

          {/* 新增任務表單 */}
          <form onSubmit={handleAddTodo} className="flex space-x-2 mb-8">
            <input
              type="text"
              value={newTodoText}
              onChange={(e) => setNewTodoText(e.target.value)}
              placeholder="輸入新的待辦事項..."
              className="flex-grow p-3 border-2 border-gray-300 rounded-lg focus:outline-none focus:border-indigo-500 text-gray-700 transition duration-150"
              aria-label="新的待辦事項輸入"
            />
            <button
              type="submit"
              className="p-3 bg-indigo-600 text-white rounded-lg shadow-md hover:bg-indigo-700 transition duration-200 transform hover:scale-105 flex items-center justify-center"
              aria-label="新增任務"
            >
              <Plus className="w-6 h-6" />
            </button>
          </form>

          {/* 待辦事項列表 */}
          <div className="space-y-3">
            {todos.length > 0 ? (
              todos.map(todo => (
                <TodoItem 
                  key={todo.id} 
                  todo={todo} 
                  onToggle={handleToggleTodo} 
                  onDelete={handleDeleteTodo} 
                />
              ))
            ) : (
              <div className="text-center p-10 bg-gray-50 rounded-lg border border-dashed border-gray-300">
                <p className="text-lg text-gray-500 font-medium">🎉 恭喜您，目前沒有任何待辦事項！</p>
                <p className="text-sm text-gray-400 mt-2">在上方輸入框中新增您的第一個任務吧。</p>
              </div>
            )}
          </div>
          
        </div>
      </div>
    );
  };

  window.onload = () => {
    if (window['lucide']) {
        ReactDOM.createRoot(document.getElementById('root')).render(<App />);
    } else {
        console.error("Lucide icons 庫未能載入。");
    }
  };
</script>

<!-- Lucide Icons 載入 (用於 React 組件) -->
<script src="https://unpkg.com/lucide@latest"></script>

</body>
</html>
