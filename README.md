# gradebook2
book for teachers
const STORAGE_KEY = "umid-gradebook";

const defaultData = {
  groups: [
    { id: "g1", name: "Frontend 101", createdAt: "2024-08-01" },
    { id: "g2", name: "Backend Pro", createdAt: "2024-08-10" },
  ],
  students: [
    {
      id: "s1",
      name: "Nodira Karimova",
      groupId: "g1",
      attendance: [
        { date: "2024-09-01", status: "kelgan" },
        { date: "2024-09-05", status: "kechikkan" },
      ],
      grades: [
        { title: "1-nazorat", score: 85, date: "2024-09-06" },
      ],
      assignments: [
        {
          id: "a1",
          title: "Portfolio sahifa",
          description: "Shaxsiy portfolioni HTML/CSS bilan yarating",
          dueDate: "2024-09-15",
          status: "kutilmoqda",
          submissionText: "",
          submittedAt: null,
        },
      ],
    },
    {
      id: "s2",
      name: "Azizbek Tursunov",
      groupId: "g2",
      attendance: [{ date: "2024-09-02", status: "kelgan" }],
      grades: [
        { title: "Backend test", score: 78, date: "2024-09-07" },
      ],
      assignments: [],
    },
  ],
};

const state = loadData();

const roleSelect = document.getElementById("roleSelect");
const studentLogin = document.getElementById("studentLogin");
const studentSelect = document.getElementById("studentSelect");
const enterBtn = document.getElementById("enterBtn");
const adminView = document.getElementById("adminView");
const studentView = document.getElementById("studentView");

const groupForm = document.getElementById("groupForm");
const groupName = document.getElementById("groupName");
const groupList = document.getElementById("groupList");
const studentForm = document.getElementById("studentForm");
const studentName = document.getElementById("studentName");
const studentGroupSelect = document.getElementById("studentGroupSelect");
const studentList = document.getElementById("studentList");

const adminStudentSelect = document.getElementById("adminStudentSelect");
const attendanceForm = document.getElementById("attendanceForm");
const attendanceDate = document.getElementById("attendanceDate");
const attendanceStatus = document.getElementById("attendanceStatus");
const attendanceList = document.getElementById("attendanceList");

const gradeForm = document.getElementById("gradeForm");
const gradeTitle = document.getElementById("gradeTitle");
const gradeScore = document.getElementById("gradeScore");
const gradeList = document.getElementById("gradeList");

const assignmentForm = document.getElementById("assignmentForm");
const assignmentTitle = document.getElementById("assignmentTitle");
const assignmentDesc = document.getElementById("assignmentDesc");
const assignmentDue = document.getElementById("assignmentDue");
const assignmentGroup = document.getElementById("assignmentGroup");
const assignmentList = document.getElementById("assignmentList");

const studentMeta = document.getElementById("studentMeta");
const studentAttendance = document.getElementById("studentAttendance");
const studentGrades = document.getElementById("studentGrades");
const studentAssignments = document.getElementById("studentAssignments");

function loadData() {
  const stored = localStorage.getItem(STORAGE_KEY);
  if (stored) {
    return JSON.parse(stored);
  }
  localStorage.setItem(STORAGE_KEY, JSON.stringify(defaultData));
  return JSON.parse(JSON.stringify(defaultData));
}

function saveData() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(state));
}

function uid(prefix) {
  return `${prefix}-${Math.random().toString(36).slice(2, 9)}`;
}

function renderGroups() {
  groupList.innerHTML = "";
  if (state.groups.length === 0) {
    groupList.innerHTML = "<li>Hali guruhlar yo'q.</li>";
  }
  state.groups.forEach((group) => {
    const li = document.createElement("li");
    const info = document.createElement("span");
    info.textContent = `${group.name} · ${group.createdAt}`;

    const action = document.createElement("button");
    action.textContent = "O'chirish";
    action.classList.add("danger-btn");
    action.addEventListener("click", () => removeGroup(group.id));

    li.append(info, action);
    groupList.appendChild(li);
  });

  const options = state.groups
    .map((group) => `<option value="${group.id}">${group.name}</option>`)
    .join("");

  const fallback = "<option value=\"\">Guruh tanlang</option>";
  studentGroupSelect.innerHTML = fallback + options;
  assignmentGroup.innerHTML = fallback + options;
  const hasGroups = state.groups.length > 0;
  studentGroupSelect.disabled = !hasGroups;
  assignmentGroup.disabled = !hasGroups;
}

function renderStudents() {
  studentList.innerHTML = "";
  if (state.students.length === 0) {
    studentList.innerHTML = "<p class=\"muted\">Hali talabalar yo'q.</p>";
  }
  state.students.forEach((student) => {
    const group = state.groups.find((item) => item.id === student.groupId);
    const card = document.createElement("div");
    card.className = "card";
    card.innerHTML = `
      <h4>${student.name}</h4>
      <p class="muted">Guruh: ${group ? group.name : "-"}</p>
      <p class="muted">Davomat: ${student.attendance.length} yozuv</p>
      <p class="muted">Baholar: ${student.grades.length} yozuv</p>
    `;
    studentList.appendChild(card);
  });

  const options = state.students
    .map((student) => `<option value="${student.id}">${student.name}</option>`)
    .join("");

  const fallback = "<option value=\"\">Talaba tanlang</option>";
  adminStudentSelect.innerHTML = fallback + options;
  studentSelect.innerHTML = fallback + options;
  const hasStudents = state.students.length > 0;
  adminStudentSelect.disabled = !hasStudents;
  studentSelect.disabled = !hasStudents;
}

function renderSelectedStudent(studentId) {
  const student = state.students.find((item) => item.id === studentId);
  if (!student) {
    attendanceList.innerHTML = "";
    gradeList.innerHTML = "";
    assignmentList.innerHTML = "";
    return;
  }

  attendanceList.innerHTML = student.attendance
    .map((item) => `<li><span>${item.date}</span><span>${item.status}</span></li>`)
    .join("");

  gradeList.innerHTML = student.grades
    .map(
      (item) =>
        `<li><span>${item.title}</span><span>${item.score}</span></li>`
    )
    .join("");

  assignmentList.innerHTML = student.assignments
    .map(
      (item) =>
        `<li><span>${item.title}</span><span>${item.dueDate}</span></li>`
    )
    .join("");
}

function removeGroup(groupId) {
  const hasStudents = state.students.some((student) => student.groupId === groupId);
  if (hasStudents) {
    alert("Guruhda talabalar bor. Avval talabalarni boshqa guruhga o'tkazing.");
    return;
  }
  state.groups = state.groups.filter((group) => group.id !== groupId);
  saveData();
  renderGroups();
  renderStudents();
}

function setupAdminEvents() {
  groupForm.addEventListener("submit", (event) => {
    event.preventDefault();
    const name = groupName.value.trim();
    if (!name) return;
    state.groups.push({
      id: uid("g"),
      name,
      createdAt: new Date().toISOString().split("T")[0],
    });
    groupName.value = "";
    saveData();
    renderGroups();
  });

  studentForm.addEventListener("submit", (event) => {
    event.preventDefault();
    const name = studentName.value.trim();
    const groupId = studentGroupSelect.value;
    if (!name || !groupId) return;
    state.students.push({
      id: uid("s"),
      name,
      groupId,
      attendance: [],
      grades: [],
      assignments: [],
    });
    studentName.value = "";
    saveData();
    renderStudents();
  });

  adminStudentSelect.addEventListener("change", (event) => {
    renderSelectedStudent(event.target.value);
  });

  attendanceForm.addEventListener("submit", (event) => {
    event.preventDefault();
    const studentId = adminStudentSelect.value;
    const student = state.students.find((item) => item.id === studentId);
    if (!student) return;
    student.attendance.unshift({
      date: attendanceDate.value,
      status: attendanceStatus.value,
    });
    attendanceDate.value = "";
    saveData();
    renderSelectedStudent(studentId);
  });

  gradeForm.addEventListener("submit", (event) => {
    event.preventDefault();
    const studentId = adminStudentSelect.value;
    const student = state.students.find((item) => item.id === studentId);
    if (!student) return;
    student.grades.unshift({
      title: gradeTitle.value.trim(),
      score: Number(gradeScore.value),
      date: new Date().toISOString().split("T")[0],
    });
    gradeTitle.value = "";
    gradeScore.value = "";
    saveData();
    renderSelectedStudent(studentId);
  });

  assignmentForm.addEventListener("submit", (event) => {
    event.preventDefault();
    const groupId = assignmentGroup.value;
    const assignment = {
      id: uid("a"),
      title: assignmentTitle.value.trim(),
      description: assignmentDesc.value.trim(),
      dueDate: assignmentDue.value,
      status: "kutilmoqda",
      submissionText: "",
      submittedAt: null,
    };
    state.students
      .filter((student) => student.groupId === groupId)
      .forEach((student) => student.assignments.unshift({ ...assignment }));

    assignmentTitle.value = "";
    assignmentDesc.value = "";
    assignmentDue.value = "";
    saveData();
    renderSelectedStudent(adminStudentSelect.value);
  });
}

function setView(role) {
  if (role === "admin") {
    adminView.classList.add("active");
    studentView.classList.remove("active");
  } else {
    adminView.classList.remove("active");
    studentView.classList.add("active");
  }
}

function renderStudentPanel(studentId) {
  const student = state.students.find((item) => item.id === studentId);
  if (!student) return;
  const group = state.groups.find((item) => item.id === student.groupId);
  studentMeta.textContent = `${student.name} · ${group ? group.name : ""}`;

  studentAttendance.innerHTML = student.attendance
    .map(
      (item) =>
        `<li><span>${item.date}</span><span>${item.status}</span></li>`
    )
    .join("");

  studentGrades.innerHTML = student.grades
    .map(
      (item) =>
        `<li><span>${item.title}</span><span>${item.score}</span></li>`
    )
    .join("");

  studentAssignments.innerHTML = "";
  student.assignments.forEach((assignment) => {
    const card = document.createElement("div");
    card.className = "card";
    const statusClass = assignment.status === "topsirildi" ? "success" : "warning";
    card.innerHTML = `
      <h4>${assignment.title}</h4>
      <p class="muted">Muddat: ${assignment.dueDate}</p>
      <p>${assignment.description || "Tavsif berilmagan"}</p>
      <span class="tag ${statusClass}">${assignment.status}</span>
    `;
    if (assignment.status !== "topsirildi") {
      const textarea = document.createElement("textarea");
      textarea.placeholder = "Yechim yoki izoh";
      textarea.rows = 2;
      textarea.value = assignment.submissionText;
      const button = document.createElement("button");
      button.textContent = "Topshirish";
      button.addEventListener("click", () => {
        assignment.status = "topsirildi";
        assignment.submissionText = textarea.value;
        assignment.submittedAt = new Date().toISOString();
        saveData();
        renderStudentPanel(studentId);
      });
      card.append(textarea, button);
    } else {
      const submitted = document.createElement("p");
      submitted.className = "muted";
      submitted.textContent = `Topshirilgan: ${new Date(
        assignment.submittedAt
      ).toLocaleString()}`;
      card.appendChild(submitted);
    }
    studentAssignments.appendChild(card);
  });
}

roleSelect.addEventListener("change", (event) => {
  if (event.target.value === "student") {
    studentLogin.classList.remove("hidden");
  } else {
    studentLogin.classList.add("hidden");
  }
});

enterBtn.addEventListener("click", () => {
  const role = roleSelect.value;
  setView(role);
  if (role === "student") {
    renderStudentPanel(studentSelect.value);
  }
});

studentSelect.addEventListener("change", (event) => {
  renderStudentPanel(event.target.value);
});

function init() {
  setView(roleSelect.value);
  renderGroups();
  renderStudents();
  setupAdminEvents();
  if (state.students.length > 0) {
    adminStudentSelect.value = state.students[0].id;
    renderSelectedStudent(state.students[0].id);
  }
}

init();
