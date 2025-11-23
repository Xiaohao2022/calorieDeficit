<script setup>
import { ref, computed } from 'vue';

// 日期选择
const currentDate = ref(new Date().toISOString().split('T')[0]);

// 体重数据
const morningWeight = ref('');
const eveningWeight = ref('');

// 饮食数据
const meals = ref([
  { type: '早餐', foods: [{ name: '', calories: '' }] },
  { type: '午餐', foods: [{ name: '', calories: '' }] },
  { type: '晚餐', foods: [{ name: '', calories: '' }] }
]);

// 运动数据
const exercises = ref([{ type: '', duration: '', caloriesBurned: '' }]);

// 基础代谢率（简化计算，实际应根据用户信息计算）
const baseMetabolism = ref(1500);

// 添加食物
function addFood(mealIndex) {
  meals.value[mealIndex].foods.push({ name: '', calories: '' });
}

// 删除食物
function removeFood(mealIndex, foodIndex) {
  if (meals.value[mealIndex].foods.length > 1) {
    meals.value[mealIndex].foods.splice(foodIndex, 1);
  }
}

// 添加运动
function addExercise() {
  exercises.value.push({ type: '', duration: '', caloriesBurned: '' });
}

// 删除运动
function removeExercise(exerciseIndex) {
  if (exercises.value.length > 1) {
    exercises.value.splice(exerciseIndex, 1);
  }
}

// 计算总摄入热量
const totalCaloriesConsumed = computed(() => {
  return meals.value.reduce((total, meal) => {
    return total + meal.foods.reduce((mealTotal, food) => {
      return mealTotal + (parseInt(food.calories) || 0);
    }, 0);
  }, 0);
});

// 计算总消耗热量
const totalCaloriesBurned = computed(() => {
  return exercises.value.reduce((total, exercise) => {
    return total + (parseInt(exercise.caloriesBurned) || 0);
  }, parseInt(baseMetabolism.value) || 0);
});

// 计算热量缺口
const calorieDeficit = computed(() => {
  return totalCaloriesBurned.value - totalCaloriesConsumed.value;
});

// 保存数据
function saveData() {
  const data = {
    date: currentDate.value,
    morningWeight: morningWeight.value,
    eveningWeight: eveningWeight.value,
    meals: meals.value,
    exercises: exercises.value,
    baseMetabolism: baseMetabolism.value
  };
  localStorage.setItem(`calorie_data_${currentDate.value}`, JSON.stringify(data));
  alert('数据已保存！');
}

// 加载数据
function loadData() {
  const savedData = localStorage.getItem(`calorie_data_${currentDate.value}`);
  if (savedData) {
    const data = JSON.parse(savedData);
    morningWeight.value = data.morningWeight || '';
    eveningWeight.value = data.eveningWeight || '';
    meals.value = data.meals || meals.value;
    exercises.value = data.exercises || exercises.value;
    baseMetabolism.value = data.baseMetabolism || baseMetabolism.value;
  }
}

// 组件挂载时加载数据
loadData();
</script>

<template>
  <div class="app-container">
    <h1>每日热量记录</h1>
    
    <!-- 日期选择 -->
    <div class="date-selector">
      <label for="date">选择日期：</label>
      <input 
        type="date" 
        id="date" 
        v-model="currentDate"
        @change="loadData"
      />
    </div>

    <div class="dashboard">
      <!-- 左侧：记录区域 -->
      <div class="record-section">
        <!-- 体重记录 -->
        <div class="weight-record">
          <h2>体重记录</h2>
          <div class="weight-inputs">
            <div class="input-group">
              <label>早上体重 (kg)：</label>
              <input type="number" v-model.number="morningWeight" step="0.1" min="30" max="200" />
            </div>
            <div class="input-group">
              <label>晚上体重 (kg)：</label>
              <input type="number" v-model.number="eveningWeight" step="0.1" min="30" max="200" />
            </div>
          </div>
        </div>

        <!-- 饮食记录 -->
        <div class="food-record">
          <h2>饮食记录</h2>
          <div class="meals">
            <div v-for="(meal, mealIndex) in meals" :key="mealIndex" class="meal">
              <h3>{{ meal.type }}</h3>
              <div v-for="(food, foodIndex) in meal.foods" :key="foodIndex" class="food-item">
                <input 
                  type="text" 
                  v-model="food.name" 
                  placeholder="食物名称"
                />
                <input 
                  type="number" 
                  v-model.number="food.calories" 
                  placeholder="热量 (kcal)"
                  min="0"
                />
                <button 
                  @click="removeFood(mealIndex, foodIndex)"
                  class="remove-btn"
                >
                  删除
                </button>
              </div>
              <button @click="addFood(mealIndex)" class="add-btn">
                添加食物
              </button>
            </div>
          </div>
        </div>

        <!-- 运动记录 -->
        <div class="exercise-record">
          <h2>运动记录</h2>
          <div v-for="(exercise, index) in exercises" :key="index" class="exercise-item">
            <input 
              type="text" 
              v-model="exercise.type" 
              placeholder="运动类型"
            />
            <input 
              type="number" 
              v-model.number="exercise.duration" 
              placeholder="时长 (分钟)"
              min="0"
            />
            <input 
              type="number" 
              v-model.number="exercise.caloriesBurned" 
              placeholder="消耗热量 (kcal)"
              min="0"
            />
            <button 
              @click="removeExercise(index)"
              class="remove-btn"
            >
              删除
            </button>
          </div>
          <button @click="addExercise" class="add-btn">
            添加运动
          </button>
        </div>

        <!-- 基础代谢设置 -->
        <div class="metabolism-setting">
          <h2>基础代谢</h2>
          <div class="input-group">
            <label>基础代谢率 (kcal)：</label>
            <input 
              type="number" 
              v-model.number="baseMetabolism" 
              min="500" 
              max="5000"
            />
          </div>
        </div>

        <button @click="saveData" class="save-btn">保存数据</button>
      </div>

      <!-- 右侧：结果显示 -->
      <div class="result-section">
        <h2>热量统计</h2>
        <div class="stats">
          <div class="stat-item">
            <span class="label">总摄入热量：</span>
            <span class="value">{{ totalCaloriesConsumed }} kcal</span>
          </div>
          <div class="stat-item">
            <span class="label">基础代谢消耗：</span>
            <span class="value">{{ baseMetabolism }} kcal</span>
          </div>
          <div class="stat-item">
            <span class="label">运动消耗热量：</span>
            <span class="value">{{ totalCaloriesBurned - baseMetabolism }} kcal</span>
          </div>
          <div class="stat-item">
            <span class="label">总消耗热量：</span>
            <span class="value">{{ totalCaloriesBurned }} kcal</span>
          </div>
          <div class="stat-item deficit" :class="{ 'positive': calorieDeficit > 0, 'negative': calorieDeficit < 0 }">
            <span class="label">热量缺口：</span>
            <span class="value">{{ calorieDeficit }} kcal</span>
          </div>
        </div>
        <div class="status-message" :class="{ 'success': calorieDeficit > 0, 'warning': calorieDeficit <= 0 }">
          <p v-if="calorieDeficit > 0">恭喜！您今天的热量缺口达标，继续保持！</p>
          <p v-else-if="calorieDeficit < 0">注意：您今天摄入的热量超过了消耗，建议增加运动或减少摄入。</p>
          <p v-else>您今天的热量摄入与消耗持平，保持均衡的饮食和运动习惯。</p>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
/* 全局样式 */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: 'Arial', sans-serif;
  line-height: 1.6;
  color: #333;
  background-color: #f5f5f5;
}

.app-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

h1 {
  text-align: center;
  margin-bottom: 30px;
  color: #2c3e50;
}

h2 {
  margin-bottom: 20px;
  color: #34495e;
  border-bottom: 2px solid #3498db;
  padding-bottom: 10px;
}

h3 {
  margin-bottom: 15px;
  color: #7f8c8d;
}

/* 日期选择器 */
.date-selector {
  text-align: center;
  margin-bottom: 30px;
}

.date-selector label {
  font-size: 18px;
  margin-right: 10px;
}

.date-selector input[type="date"] {
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

/* 仪表板布局 */
.dashboard {
  display: flex;
  gap: 30px;
}

.record-section {
  flex: 2;
  background: white;
  padding: 25px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.result-section {
  flex: 1;
  background: white;
  padding: 25px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

/* 输入组 */
.input-group {
  margin-bottom: 15px;
}

.input-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: 500;
}

.input-group input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
}

/* 体重记录 */
.weight-inputs {
  display: flex;
  gap: 20px;
  margin-bottom: 20px;
}

.weight-inputs .input-group {
  flex: 1;
}

/* 食物和运动项目 */
.food-item, .exercise-item {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
  align-items: center;
}

.food-item input, .exercise-item input {
  flex: 1;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
}

.meal {
  margin-bottom: 30px;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 8px;
}

/* 按钮样式 */
.add-btn, .remove-btn, .save-btn {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: background-color 0.3s;
}

.add-btn {
  background-color: #3498db;
  color: white;
  margin-top: 10px;
}

.add-btn:hover {
  background-color: #2980b9;
}

.remove-btn {
  background-color: #e74c3c;
  color: white;
  min-width: 60px;
}

.remove-btn:hover {
  background-color: #c0392b;
}

.save-btn {
  background-color: #2ecc71;
  color: white;
  padding: 12px 30px;
  font-size: 16px;
  width: 100%;
  margin-top: 30px;
}

.save-btn:hover {
  background-color: #27ae60;
}

/* 统计结果 */
.stats {
  margin-bottom: 30px;
}

.stat-item {
  display: flex;
  justify-content: space-between;
  margin-bottom: 15px;
  padding: 15px;
  background-color: #f9f9f9;
  border-radius: 4px;
}

.stat-item.deficit {
  font-weight: bold;
}

.stat-item.deficit.positive {
  background-color: #d4edda;
  color: #155724;
}

.stat-item.deficit.negative {
  background-color: #f8d7da;
  color: #721c24;
}

.stat-item .label {
  font-weight: 500;
}

.stat-item .value {
  font-size: 18px;
  font-weight: bold;
}

/* 状态消息 */
.status-message {
  padding: 20px;
  border-radius: 4px;
  text-align: center;
  font-size: 16px;
}

.status-message.success {
  background-color: #d4edda;
  color: #155724;
  border: 1px solid #c3e6cb;
}

.status-message.warning {
  background-color: #fff3cd;
  color: #856404;
  border: 1px solid #ffeaa7;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .dashboard {
    flex-direction: column;
  }
  
  .weight-inputs {
    flex-direction: column;
  }
  
  .food-item, .exercise-item {
    flex-direction: column;
  }
  
  .food-item input, .exercise-item input {
    width: 100%;
  }
}
</style>
