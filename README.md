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
  <title>極簡待辦清單 (Local Storage)</title>
  
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://unpkg.com/react@18/umd/react.development.js"></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  
  <style>

    body {
      min-height: 100vh;
    }
  </style>
</head>
<body class="bg-gray-100">

<div id="root"></div>

<script type="text/babel">
  const { useState, useEffect, useCallback } = React;
  const { ListChecks, Plus, Trash2, CheckCircle, Circle } = window['lucide']; 
  const LOCAL_STORAGE_KEY = 'simpleTodoListTasks';
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
    } catch (error) {
      console.error("寫入 Local Storage 失敗:", error);
    }
  };

  const TodoItem = React.memo(({ todo, onToggle, onDelete }) => (
    <div className={`flex items-center p-3 border border-gray-300 rounded mb-2 
                    ${todo.completed ? 'bg-gray-200' : 'bg-white'}`}>
      
      {/* 標記完成按鈕 */}
      <button 
        onClick={() => onToggle(todo.id)}
        className="w-6 h-6 flex items-center justify-center text-gray-500 mr-3 flex-shrink-0"
        aria-label={todo.completed ? '標記為未完成' : '標記為完成'}
      >
        {todo.completed 
          ? <CheckCircle className="w-5 h-5 text-green-600" /> 
          : <Circle className="w-5 h-5" />
        }
      </button>

      {/* 任務文字 */}
      <span className={`flex-grow text-lg break-words
                       ${todo.completed ? 'line-through text-gray-500' : 'text-gray-800'}`}>
        {todo.text}
      </span>

      {/* 刪除按鈕 */}
      <button 
        onClick={() => onDelete(todo.id)}
        className="ml-4 p-1 text-red-500 hover:text-red-700 flex-shrink-0"
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

      setTodos(prevTodos => [newTodo, ...prevTodos]);
      setNewTodoText('');
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

    return (
      <div className="flex flex-col items-center p-4">
        <div className="w-full max-w-xl bg-white p-6 rounded-lg border border-gray-300">
          
          {/* 標題區 */}
          <h1 className="text-2xl font-bold text-center text-gray-800 mb-6 flex items-center justify-center">
            <ListChecks className="w-6 h-6 mr-2 text-gray-600" />
            待辦清單 (Todo List)
          </h1>

          {/* 新增任務表單 */}
          <form onSubmit={handleAddTodo} className="flex space-x-2 mb-6">
            <input
              type="text"
              value={newTodoText}
              onChange={(e) => setNewTodoText(e.target.value)}
              placeholder="輸入新的待辦事項..."
              className="flex-grow p-2 border border-gray-400 rounded focus:outline-none focus:border-blue-500"
              aria-label="新的待辦事項輸入"
            />
            <button
              type="submit"
              className="p-2 bg-blue-500 text-white rounded hover:bg-blue-600 transition"
              aria-label="新增任務"
            >
              <Plus className="w-6 h-6" />
            </button>
          </form>

          {/* 待辦事項列表 */}
          <div className="space-y-3">
            {todos.length > 0 ? (
              // 使用基本的 div 包裹列表，無額外樣式
              <div>
                {todos.map(todo => (
                  <TodoItem 
                    key={todo.id} 
                    todo={todo} 
                    onToggle={handleToggleTodo} 
                    onDelete={handleDeleteTodo} 
                  />
                ))}
              </div>
            ) : (
              <div className="text-center p-4 bg-gray-50 border border-dashed border-gray-300 rounded">
                <p className="text-gray-500">目前沒有任何任務。</p>
              </div>
            )}
          </div>
          
        </div>
      </div>
    );
  };

  window.onload = () => {
    // 確保 Lucide 圖標庫已載入
    if (window['lucide']) {
        ReactDOM.createRoot(document.getElementById('root')).render(<App />);
    } else {
        console.error("Lucide icons 庫未能載入。");
    }
  };
</script>

<!-- Lucide Icons 載入 -->
<script src="https://unpkg.com/lucide@latest"></script>

</body>
</html>
