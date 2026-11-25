[Output] : (https://sites.google.com/d/1_sLuug3LPRUERYNbFJRZoUQ_hHxbsfRl/p/1IzA_RrzxMwZ157fkJZ1cKsVV2kBankRw/edit?pli=1)
Code >>
import React, { useState, useEffect, useMemo, useRef } from 'react';
import {
  GraduationCap,
  Users,
  Building2,
  Award,
  UserPlus,
  Edit2,
  Trash2,
  Eye,
  Search,
  Filter,
  X,
  CheckCircle2,
  AlertCircle,
  Mail,
  Phone,
  Hash,
  BookOpen,
  Calendar,
  RotateCcw,
} from 'lucide-react';

// ============================================================================
// Constants & Initial Sample Data
// ============================================================================

const STORAGE_KEY = 'student_management_app_records_v1';

const DEPARTMENTS = [
  'Computer Science',
  'Electronics',
  'Mechanical',
  'Civil',
  'Business',
  'Data Science',
];

const YEARS = ['1st Year', '2nd Year', '3rd Year', '4th Year'];

const INITIAL_STUDENTS = [
  {
    id: 'rec-1001',
    fullName: 'Aarav Sharma',
    studentId: 'CS2024001',
    email: 'aarav.sharma@university.edu',
    phone: '9876543210',
    department: 'Computer Science',
    year: '3rd Year',
    gpa: 3.85,
  },
  {
    id: 'rec-1002',
    fullName: 'Maya Lin',
    studentId: 'DS2024014',
    email: 'maya.lin@university.edu',
    phone: '9123456789',
    department: 'Data Science',
    year: '2nd Year',
    gpa: 3.92,
  },
  {
    id: 'rec-1003',
    fullName: 'Liam O’Connor',
    studentId: 'EC2023042',
    email: 'liam.oconnor@university.edu',
    phone: '8765432109',
    department: 'Electronics',
    year: '4th Year',
    gpa: 3.48,
  },
  {
    id: 'rec-1004',
    fullName: 'Priya Nair',
    studentId: 'BS2025008',
    email: 'priya.nair@university.edu',
    phone: '9988776655',
    department: 'Business',
    year: '1st Year',
    gpa: 3.64,
  },
];

const EMPTY_FORM_STATE = {
  fullName: '',
  studentId: '',
  email: '',
  phone: '',
  department: '',
  year: '',
  gpa: '',
};

// Helper for department badge styling
const getDepartmentBadgeStyle = (department) => {
  const styles = {
    'Computer Science': 'bg-indigo-50 text-indigo-700 border-indigo-200',
    'Data Science': 'bg-violet-50 text-violet-700 border-violet-200',
    Electronics: 'bg-sky-50 text-sky-700 border-sky-200',
    Mechanical: 'bg-amber-50 text-amber-700 border-amber-200',
    Civil: 'bg-emerald-50 text-emerald-700 border-emerald-200',
    Business: 'bg-rose-50 text-rose-700 border-rose-200',
  };
  return styles[department] || 'bg-slate-100 text-slate-700 border-slate-200';
};

// Helper for GPA badge styling
const getGpaBadgeStyle = (gpa) => {
  const numericGpa = Number(gpa);
  if (numericGpa >= 3.7) return 'bg-emerald-50 text-emerald-700 ring-emerald-600/20';
  if (numericGpa >= 3.0) return 'bg-blue-50 text-blue-700 ring-blue-600/20';
  if (numericGpa >= 2.0) return 'bg-amber-50 text-amber-700 ring-amber-600/20';
  return 'bg-rose-50 text-rose-700 ring-rose-600/20';
};

// ============================================================================
// Component 1: Summary Stats Cards
// ============================================================================

const StatsCards = ({ students }) => {
  const totalStudents = students.length;

  const activeDepartments = useMemo(() => {
    const unique = new Set(students.map((s) => s.department));
    return unique.size;
  }, [students]);

  const averageGpa = useMemo(() => {
    if (students.length === 0) return '0.00';
    const sum = students.reduce((acc, s) => acc + Number(s.gpa || 0), 0);
    return (sum / students.length).toFixed(2);
  }, [students]);

  const cards = [
    {
      label: 'Total Students',
      value: totalStudents,
      subtext: totalStudents === 1 ? '1 registered student' : `${totalStudents} registered students`,
      icon: Users,
      iconBg: 'bg-indigo-50 text-indigo-600',
    },
    {
      label: 'Departments',
      value: activeDepartments,
      subtext: `Out of ${DEPARTMENTS.length} offered courses`,
      icon: Building2,
      iconBg: 'bg-sky-50 text-sky-600',
    },
    {
      label: 'Average GPA',
      value: averageGpa,
      subtext: 'On a 4.00 grading scale',
      icon: Award,
      iconBg: 'bg-emerald-50 text-emerald-600',
    },
  ];

  return (
    <section aria-label="Student Summary Statistics" className="grid grid-cols-1 sm:grid-cols-3 gap-4">
      {cards.map((card) => {
        const IconComponent = card.icon;
        return (
          <div
            key={card.label}
            className="bg-white rounded-2xl border border-slate-200/80 p-5 shadow-sm hover:shadow-md transition-shadow duration-200 flex items-center justify-between"
          >
            <div>
              <p className="text-sm font-medium text-slate-500">{card.label}</p>
              <p className="mt-1.5 text-3xl font-bold tracking-tight text-slate-900">
                {card.value}
              </p>
              <p className="mt-1 text-xs text-slate-400">{card.subtext}</p>
            </div>
            <div className={`p-3.5 rounded-xl ${card.iconBg}`}>
              <IconComponent className="w-6 h-6" />
            </div>
          </div>
        );
      })}
    </section>
  );
};

// ============================================================================
// Component 2: Student Registration & Edit Form
// ============================================================================

const StudentForm = ({
  students,
  editingStudent,
  onSaveStudent,
  onCancelEdit,
  formContainerRef,
}) => {
  const [formData, setFormData] = useState(EMPTY_FORM_STATE);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});

  const isEditMode = Boolean(editingStudent);

  // Sync form state whenever editingStudent changes
  useEffect(() => {
    if (editingStudent) {
      setFormData({
        fullName: editingStudent.fullName,
        studentId: editingStudent.studentId,
        email: editingStudent.email,
        phone: editingStudent.phone,
        department: editingStudent.department,
        year: editingStudent.year,
        gpa: String(editingStudent.gpa),
      });
      setErrors({});
      setTouched({});
    } else {
      setFormData(EMPTY_FORM_STATE);
      setErrors({});
      setTouched({});
    }
  }, [editingStudent]);

  // Comprehensive validation logic
  const validate = (data) => {
    const newErrors = {};

    // Full Name
    const trimmedName = data.fullName.trim();
    if (!trimmedName) {
      newErrors.fullName = 'Full name is required.';
    } else if (trimmedName.length < 2) {
      newErrors.fullName = 'Full name must be at least 2 characters.';
    }

    // Student ID / Roll Number + Uniqueness Check
    const trimmedId = data.studentId.trim();
    if (!trimmedId) {
      newErrors.studentId = 'Student ID / Roll Number is required.';
    } else {
      const duplicate = students.some(
        (student) =>
          student.studentId.trim().toLowerCase() === trimmedId.toLowerCase() &&
          (!editingStudent || student.id !== editingStudent.id)
      );
      if (duplicate) {
        newErrors.studentId = 'This Student ID is already registered.';
      }
    }

    // Email format
    const trimmedEmail = data.email.trim();
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!trimmedEmail) {
      newErrors.email = 'Email address is required.';
    } else if (!emailRegex.test(trimmedEmail)) {
      newErrors.email = 'Please enter a valid email address.';
    }

    // Phone Number (exact 10 digits)
    const trimmedPhone = data.phone.trim();
    const phoneRegex = /^\d{10}$/;
    if (!trimmedPhone) {
      newErrors.phone = 'Phone number is required.';
    } else if (!phoneRegex.test(trimmedPhone)) {
      newErrors.phone = 'Phone number must be exactly 10 digits.';
    }

    // Department
    if (!data.department) {
      newErrors.department = 'Please select a department.';
    }

    // Year / Semester
    if (!data.year) {
      newErrors.year = 'Please select the academic year.';
    }

    // GPA (0.0 to 4.0)
    if (data.gpa === '' || data.gpa === null || data.gpa === undefined) {
      newErrors.gpa = 'GPA is required.';
    } else {
      const numericGpa = Number(data.gpa);
      if (Number.isNaN(numericGpa) || numericGpa < 0 || numericGpa > 4) {
        newErrors.gpa = 'GPA must be a number between 0.0 and 4.0.';
      }
    }

    return newErrors;
  };

  const handleChange = (e) => {
    const { name, value } = e.target;
    // Allow only numeric input for phone field to improve UX
    const nextValue = name === 'phone' ? value.replace(/\D/g, '').slice(0, 10) : value;
    const updatedData = { ...formData, [name]: nextValue };
    setFormData(updatedData);

    // Live re-validation for fields already touched
    if (touched[name]) {
      const validationResult = validate(updatedData);
      setErrors((prev) => ({ ...prev, [name]: validationResult[name] }));
    }
  };

  const handleBlur = (e) => {
    const { name } = e.target;
    setTouched((prev) => ({ ...prev, [name]: true }));
    const validationResult = validate(formData);
    setErrors((prev) => ({ ...prev, [name]: validationResult[name] }));
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    const validationErrors = validate(formData);
    setErrors(validationErrors);
    setTouched({
      fullName: true,
      studentId: true,
      email: true,
      phone: true,
      department: true,
      year: true,
      gpa: true,
    });

    if (Object.keys(validationErrors).length > 0) {
      return;
    }

    const payload = {
      id: editingStudent ? editingStudent.id : `rec-${Date.now()}`,
      fullName: formData.fullName.trim(),
      studentId: formData.studentId.trim().toUpperCase(),
      email: formData.email.trim().toLowerCase(),
      phone: formData.phone.trim(),
      department: formData.department,
      year: formData.year,
      gpa: Number(Number(formData.gpa).toFixed(2)),
    };

    onSaveStudent(payload, isEditMode);

    if (!isEditMode) {
      setFormData(EMPTY_FORM_STATE);
      setErrors({});
      setTouched({});
    }
  };

  const inputBaseClass = (hasError) =>
    `w-full rounded-xl border px-3.5 py-2.5 text-sm text-slate-900 placeholder-slate-400 transition-colors focus:outline-none focus:ring-2 ${
      hasError
        ? 'border-rose-400 bg-rose-50/40 focus:border-rose-500 focus:ring-rose-200'
        : 'border-slate-200 bg-white focus:border-indigo-500 focus:ring-indigo-100'
    }`;

  return (
    <div
      ref={formContainerRef}
      className={`bg-white rounded-2xl border shadow-sm transition-all duration-200 ${
        isEditMode ? 'border-indigo-300 ring-2 ring-indigo-100' : 'border-slate-200/80'
      }`}
    >
      {/* Form Header */}
      <div className="px-6 py-5 border-b border-slate-100 flex items-center justify-between">
        <div className="flex items-center gap-3">
          <div
            className={`p-2.5 rounded-xl ${
              isEditMode ? 'bg-amber-50 text-amber-600' : 'bg-indigo-50 text-indigo-600'
            }`}
          >
            {isEditMode ? <Edit2 className="w-5 h-5" /> : <UserPlus className="w-5 h-5" />}
          </div>
          <div>
            <h2 className="text-base font-semibold text-slate-900">
              {isEditMode ? 'Edit Student Details' : 'Register New Student'}
            </h2>
            <p className="text-xs text-slate-500">
              {isEditMode
                ? `Updating record for ${editingStudent.fullName}`
                : 'Fill in the details below to enroll a student'}
            </p>
          </div>
        </div>
        {isEditMode && (
          <span className="inline-flex items-center rounded-full bg-amber-50 px-2.5 py-1 text-xs font-medium text-amber-700 border border-amber-200">
            Edit Mode
          </span>
        )}
      </div>

      {/* Form Body */}
      <form onSubmit={handleSubmit} noValidate className="p-6 space-y-4">
        {/* Full Name */}
        <div>
          <label htmlFor="fullName" className="block text-xs font-semibold text-slate-700 mb-1.5">
            Full Name <span className="text-rose-500">*</span>
          </label>
          <input
            id="fullName"
            name="fullName"
            type="text"
            placeholder="e.g., Sophia Patel"
            value={formData.fullName}
            onChange={handleChange}
            onBlur={handleBlur}
            className={inputBaseClass(Boolean(errors.fullName))}
          />
          {errors.fullName && (
            <p className="mt-1 text-xs text-rose-600 flex items-center gap-1">
              <AlertCircle className="w-3.5 h-3.5 shrink-0" />
              {errors.fullName}
            </p>
          )}
        </div>

        {/* Student ID & Phone Number */}
        <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">
          <div>
            <label htmlFor="studentId" className="block text-xs font-semibold text-slate-700 mb-1.5">
              Student ID / Roll No. <span className="text-rose-500">*</span>
            </label>
            <input
              id="studentId"
              name="studentId"
              type="text"
              placeholder="e.g., CS2025012"
              value={formData.studentId}
              onChange={handleChange}
              onBlur={handleBlur}
              className={inputBaseClass(Boolean(errors.studentId))}
            />
            {errors.studentId && (
              <p className="mt-1 text-xs text-rose-600 flex items-center gap-1">
                <AlertCircle className="w-3.5 h-3.5 shrink-0" />
                {errors.studentId}
              </p>
            )}
          </div>

          <div>
            <label htmlFor="phone" className="block text-xs font-semibold text-slate-700 mb-1.5">
              Phone Number (10 digits) <span className="text-rose-500">*</span>
            </label>
            <input
              id="phone"
              name="phone"
              type="tel"
              inputMode="numeric"
              placeholder="e.g., 9876543210"
              value={formData.phone}
              onChange={handleChange}
              onBlur={handleBlur}
              className={inputBaseClass(Boolean(errors.phone))}
            />
            {errors.phone && (
              <p className="mt-1 text-xs text-rose-600 flex items-center gap-1">
                <AlertCircle className="w-3.5 h-3.5 shrink-0" />
                {errors.phone}
              </p>
            )}
          </div>
        </div>

        {/* Email */}
        <div>
          <label htmlFor="email" className="block text-xs font-semibold text-slate-700 mb-1.5">
            Email Address <span className="text-rose-500">*</span>
          </label>
          <input
            id="email"
            name="email"
            type="email"
            placeholder="e.g., sophia.patel@university.edu"
            value={formData.email}
            onChange={handleChange}
            onBlur={handleBlur}
            className={inputBaseClass(Boolean(errors.email))}
          />
          {errors.email && (
            <p className="mt-1 text-xs text-rose-600 flex items-center gap-1">
              <AlertCircle className="w-3.5 h-3.5 shrink-0" />
              {errors.email}
            </p>
          )}
        </div>

        {/* Department */}
        <div>
          <label htmlFor="department" className="block text-xs font-semibold text-slate-700 mb-1.5">
            Department / Course <span className="text-rose-500">*</span>
          </label>
          <select
            id="department"
            name="department"
            value={formData.department}
            onChange={handleChange}
            onBlur={handleBlur}
            className={inputBaseClass(Boolean(errors.department))}
          >
            <option value="">Select a department...</option>
            {DEPARTMENTS.map((dept) => (
              <option key={dept} value={dept}>
                {dept}
              </option>
            ))}
          </select>
          {errors.department && (
            <p className="mt-1 text-xs text-rose-600 flex items-center gap-1">
              <AlertCircle className="w-3.5 h-3.5 shrink-0" />
              {errors.department}
            </p>
          )}
        </div>

        {/* Year & GPA */}
        <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">
          <div>
            <label htmlFor="year" className="block text-xs font-semibold text-slate-700 mb-1.5">
              Year / Semester <span className="text-rose-500">*</span>
            </label>
            <select
              id="year"
              name="year"
              value={formData.year}
              onChange={handleChange}
              onBlur={handleBlur}
              className={inputBaseClass(Boolean(errors.year))}
            >
              <option value="">Select year...</option>
              {YEARS.map((yr) => (
                <option key={yr} value={yr}>
                  {yr}
                </option>
              ))}
            </select>
            {errors.year && (
              <p className="mt-1 text-xs text-rose-600 flex items-center gap-1">
                <AlertCircle className="w-3.5 h-3.5 shrink-0" />
                {errors.year}
              </p>
            )}
          </div>

          <div>
            <label htmlFor="gpa" className="block text-xs font-semibold text-slate-700 mb-1.5">
              GPA (0.0 – 4.0) <span className="text-rose-500">*</span>
            </label>
            <input
              id="gpa"
              name="gpa"
              type="number"
              step="0.01"
              min="0"
              max="4"
              placeholder="e.g., 3.75"
              value={formData.gpa}
              onChange={handleChange}
              onBlur={handleBlur}
              className={inputBaseClass(Boolean(errors.gpa))}
            />
            {errors.gpa && (
              <p className="mt-1 text-xs text-rose-600 flex items-center gap-1">
                <AlertCircle className="w-3.5 h-3.5 shrink-0" />
                {errors.gpa}
              </p>
            )}
          </div>
        </div>

        {/* Action Buttons */}
        <div className="pt-2 flex flex-col sm:flex-row gap-2.5">
          <button
            type="submit"
            className="flex-1 inline-flex items-center justify-center gap-2 rounded-xl bg-indigo-600 px-4 py-2.5 text-sm font-semibold text-white shadow-sm hover:bg-indigo-700 active:bg-indigo-800 transition-colors cursor-pointer"
          >
            {isEditMode ? (
              <>
                <CheckCircle2 className="w-4 h-4" />
                Update Student
              </>
            ) : (
              <>
                <UserPlus className="w-4 h-4" />
                Register Student
              </>
            )}
          </button>

          {isEditMode && (
            <button
              type="button"
              onClick={onCancelEdit}
              className="inline-flex items-center justify-center gap-1.5 rounded-xl border border-slate-200 bg-white px-4 py-2.5 text-sm font-semibold text-slate-700 hover:bg-slate-50 active:bg-slate-100 transition-colors cursor-pointer"
            >
              <X className="w-4 h-4" />
              Cancel
            </button>
          )}
        </div>
      </form>
    </div>
  );
};

// ============================================================================
// Component 3: Search and Filter Bar
// ============================================================================

const SearchFilterBar = ({
  searchQuery,
  onSearchChange,
  departmentFilter,
  onDepartmentFilterChange,
  totalCount,
  filteredCount,
}) => {
  const hasActiveFilters = searchQuery.trim() !== '' || departmentFilter !== 'ALL';

  const handleClear = () => {
    onSearchChange('');
    onDepartmentFilterChange('ALL');
  };

  return (
    <div className="bg-white rounded-2xl border border-slate-200/80 p-4 shadow-sm space-y-3">
      <div className="flex flex-col sm:flex-row gap-3">
        {/* Search Input */}
        <div className="relative flex-1">
          <Search className="w-4 h-4 text-slate-400 absolute left-3.5 top-1/2 -translate-y-1/2 pointer-events-none" />
          <input
            type="text"
            value={searchQuery}
            onChange={(e) => onSearchChange(e.target.value)}
            placeholder="Search by name, Student ID, email, or department..."
            aria-label="Search students"
            className="w-full rounded-xl border border-slate-200 bg-slate-50/50 pl-10 pr-9 py-2.5 text-sm text-slate-900 placeholder-slate-400 focus:bg-white focus:border-indigo-500 focus:outline-none focus:ring-2 focus:ring-indigo-100 transition-colors"
          />
          {searchQuery && (
            <button
              type="button"
              onClick={() => onSearchChange('')}
              aria-label="Clear search"
              className="absolute right-2.5 top-1/2 -translate-y-1/2 p-1 rounded-lg text-slate-400 hover:text-slate-600 hover:bg-slate-100 cursor-pointer"
            >
              <X className="w-3.5 h-3.5" />
            </button>
          )}
        </div>

        {/* Department Filter Dropdown */}
        <div className="relative sm:w-56">
          <Filter className="w-4 h-4 text-slate-400 absolute left-3.5 top-1/2 -translate-y-1/2 pointer-events-none" />
          <select
            value={departmentFilter}
            onChange={(e) => onDepartmentFilterChange(e.target.value)}
            aria-label="Filter by department"
            className="w-full rounded-xl border border-slate-200 bg-slate-50/50 pl-10 pr-4 py-2.5 text-sm text-slate-700 focus:bg-white focus:border-indigo-500 focus:outline-none focus:ring-2 focus:ring-indigo-100 transition-colors cursor-pointer"
          >
            <option value="ALL">All Departments</option>
            {DEPARTMENTS.map((dept) => (
              <option key={dept} value={dept}>
                {dept}
              </option>
            ))}
          </select>
        </div>
      </div>

      {/* Results Count & Clear Filters */}
      <div className="flex items-center justify-between text-xs text-slate-500 pt-0.5">
        <span>
          Showing <strong className="text-slate-800">{filteredCount}</strong> of{' '}
          <strong className="text-slate-800">{totalCount}</strong>{' '}
          {totalCount === 1 ? 'student' : 'students'}
        </span>
        {hasActiveFilters && (
          <button
            type="button"
            onClick={handleClear}
            className="text-indigo-600 hover:text-indigo-700 font-medium inline-flex items-center gap-1 cursor-pointer"
          >
            <X className="w-3.5 h-3.5" />
            Clear filters
          </button>
        )}
      </div>
    </div>
  );
};

// ============================================================================
// Component 4: Responsive Student List (Table on Desktop, Cards on Mobile)
// ============================================================================

const StudentList = ({
  students,
  editingStudentId,
  onViewStudent,
  onEditStudent,
  onRequestDelete,
  hasFilters,
}) => {
  if (students.length === 0) {
    return (
      <div className="bg-white rounded-2xl border border-slate-200/80 p-10 text-center shadow-sm">
        <div className="mx-auto w-12 h-12 rounded-2xl bg-slate-100 flex items-center justify-center text-slate-400 mb-3">
          <Users className="w-6 h-6" />
        </div>
        <h3 className="text-base font-semibold text-slate-900">
          {hasFilters ? 'No matching students found' : 'No students registered yet'}
        </h3>
        <p className="mt-1 text-sm text-slate-500 max-w-sm mx-auto">
          {hasFilters
            ? 'Try adjusting your search keywords or selecting a different department filter.'
            : 'Use the registration form to add your first student record to the directory.'}
        </p>
      </div>
    );
  }

  return (
    <div className="space-y-4">
      {/* Desktop Table View */}
      <div className="hidden md:block bg-white rounded-2xl border border-slate-200/80 shadow-sm overflow-hidden">
        <div className="overflow-x-auto">
          <table className="w-full text-left border-collapse">
            <thead>
              <tr className="bg-slate-50/80 border-b border-slate-200/80 text-xs font-semibold uppercase tracking-wider text-slate-500">
                <th className="py-3.5 pl-5 pr-3">Student</th>
                <th className="py-3.5 px-3">Student ID</th>
                <th className="py-3.5 px-3">Department</th>
                <th className="py-3.5 px-3">Year</th>
                <th className="py-3.5 px-3">GPA</th>
                <th className="py-3.5 pl-3 pr-5 text-right">Actions</th>
              </tr>
            </thead>
            <tbody className="divide-y divide-slate-100 text-sm">
              {students.map((student) => {
                const isBeingEdited = editingStudentId === student.id;
                return (
                  <tr
                    key={student.id}
                    className={`transition-colors ${
                      isBeingEdited ? 'bg-indigo-50/60' : 'hover:bg-slate-50/80'
                    }`}
                  >
                    <td className="py-3.5 pl-5 pr-3">
                      <div className="font-semibold text-slate-900">{student.fullName}</div>
                      <div className="text-xs text-slate-500 truncate max-w-[210px]">
                        {student.email}
                      </div>
                    </td>
                    <td className="py-3.5 px-3">
                      <span className="font-mono text-xs font-semibold text-slate-700 bg-slate-100 px-2 py-1 rounded-md">
                        {student.studentId}
                      </span>
                    </td>
                    <td className="py-3.5 px-3">
                      <span
                        className={`inline-flex items-center rounded-full border px-2.5 py-0.5 text-xs font-medium ${getDepartmentBadgeStyle(
                          student.department
                        )}`}
                      >
                        {student.department}
                      </span>
                    </td>
                    <td className="py-3.5 px-3 text-slate-600 whitespace-nowrap">
                      {student.year}
                    </td>
                    <td className="py-3.5 px-3">
                      <span
                        className={`inline-flex items-center rounded-lg px-2 py-0.5 text-xs font-bold ring-1 ring-inset ${getGpaBadgeStyle(
                          student.gpa
                        )}`}
                      >
                        {Number(student.gpa).toFixed(2)}
                      </span>
                    </td>
                    <td className="py-3.5 pl-3 pr-5 text-right whitespace-nowrap">
                      <div className="inline-flex items-center gap-1">
                        <button
                          type="button"
                          onClick={() => onViewStudent(student)}
                          title="View details"
                          aria-label={`View details for ${student.fullName}`}
                          className="p-2 rounded-lg text-slate-500 hover:text-indigo-600 hover:bg-indigo-50 transition-colors cursor-pointer"
                        >
                          <Eye className="w-4 h-4" />
                        </button>
                        <button
                          type="button"
                          onClick={() => onEditStudent(student)}
                          title="Edit student"
                          aria-label={`Edit ${student.fullName}`}
                          className="p-2 rounded-lg text-slate-500 hover:text-amber-600 hover:bg-amber-50 transition-colors cursor-pointer"
                        >
                          <Edit2 className="w-4 h-4" />
                        </button>
                        <button
                          type="button"
                          onClick={() => onRequestDelete(student)}
                          title="Delete student"
                          aria-label={`Delete ${student.fullName}`}
                          className="p-2 rounded-lg text-slate-500 hover:text-rose-600 hover:bg-rose-50 transition-colors cursor-pointer"
                        >
                          <Trash2 className="w-4 h-4" />
                        </button>
                      </div>
                    </td>
                  </tr>
                );
              })}
            </tbody>
          </table>
        </div>
      </div>

      {/* Mobile Card View */}
      <div className="grid grid-cols-1 gap-3.5 md:hidden">
        {students.map((student) => {
          const isBeingEdited = editingStudentId === student.id;
          return (
            <div
              key={student.id}
              className={`bg-white rounded-2xl border p-4 shadow-sm space-y-3 ${
                isBeingEdited ? 'border-indigo-400 ring-2 ring-indigo-100' : 'border-slate-200/80'
              }`}
            >
              <div className="flex items-start justify-between gap-2">
                <div className="min-w-0">
                  <h3 className="text-base font-semibold text-slate-900 truncate">
                    {student.fullName}
                  </h3>
                  <p className="text-xs text-slate-500 truncate">{student.email}</p>
                </div>
                <span
                  className={`shrink-0 inline-flex items-center rounded-lg px-2.5 py-1 text-xs font-bold ring-1 ring-inset ${getGpaBadgeStyle(
                    student.gpa
                  )}`}
                >
                  GPA {Number(student.gpa).toFixed(2)}
                </span>
              </div>

              <div className="flex flex-wrap items-center gap-2 text-xs">
                <span className="font-mono font-semibold text-slate-700 bg-slate-100 px-2 py-0.5 rounded-md">
                  {student.studentId}
                </span>
                <span
                  className={`inline-flex items-center rounded-full border px-2.5 py-0.5 font-medium ${getDepartmentBadgeStyle(
                    student.department
                  )}`}
                >
                  {student.department}
                </span>
                <span className="text-slate-600 bg-slate-50 border border-slate-200 px-2 py-0.5 rounded-full">
                  {student.year}
                </span>
              </div>

              <div className="pt-2 border-t border-slate-100 flex items-center justify-end gap-2">
                <button
                  type="button"
                  onClick={() => onViewStudent(student)}
                  className="flex-1 inline-flex items-center justify-center gap-1.5 rounded-xl border border-slate-200 bg-white px-3 py-2 text-xs font-semibold text-slate-700 hover:bg-slate-50 cursor-pointer"
                >
                  <Eye className="w-3.5 h-3.5" />
                  Details
                </button>
                <button
                  type="button"
                  onClick={() => onEditStudent(student)}
                  className="flex-1 inline-flex items-center justify-center gap-1.5 rounded-xl border border-amber-200 bg-amber-50/70 px-3 py-2 text-xs font-semibold text-amber-700 hover:bg-amber-100/70 cursor-pointer"
                >
                  <Edit2 className="w-3.5 h-3.5" />
                  Edit
                </button>
                <button
                  type="button"
                  onClick={() => onRequestDelete(student)}
                  className="flex-1 inline-flex items-center justify-center gap-1.5 rounded-xl border border-rose-200 bg-rose-50/70 px-3 py-2 text-xs font-semibold text-rose-700 hover:bg-rose-100/70 cursor-pointer"
                >
                  <Trash2 className="w-3.5 h-3.5" />
                  Delete
                </button>
              </div>
            </div>
          );
        })}
      </div>
    </div>
  );
};

// ============================================================================
// Component 5: View Student Details Modal
// ============================================================================

const StudentDetailsModal = ({ student, onClose, onEdit }) => {
  useEffect(() => {
    if (!student) return;
    const handleKeyDown = (e) => {
      if (e.key === 'Escape') onClose();
    };
    window.addEventListener('keydown', handleKeyDown);
    return () => window.removeEventListener('keydown', handleKeyDown);
  }, [student, onClose]);

  if (!student) return null;

  const detailItems = [
    { label: 'Student ID / Roll No.', value: student.studentId, icon: Hash },
    { label: 'Email Address', value: student.email, icon: Mail },
    { label: 'Phone Number', value: student.phone, icon: Phone },
    { label: 'Department / Course', value: student.department, icon: BookOpen },
    { label: 'Year / Semester', value: student.year, icon: Calendar },
    { label: 'Current GPA', value: `${Number(student.gpa).toFixed(2)} / 4.00`, icon: Award },
  ];

  return (
    <div
      className="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-900/50 backdrop-blur-xs"
      role="dialog"
      aria-modal="true"
      aria-labelledby="student-details-title"
      onClick={onClose}
    >
      <div
        className="w-full max-w-lg bg-white rounded-2xl shadow-xl border border-slate-200 overflow-hidden"
        onClick={(e) => e.stopPropagation()}
      >
        {/* Modal Header */}
        <div className="px-6 py-5 bg-slate-50 border-b border-slate-200/80 flex items-start justify-between gap-4">
          <div className="flex items-center gap-3.5">
            <div className="w-12 h-12 rounded-2xl bg-indigo-600 text-white font-bold text-lg flex items-center justify-center shadow-sm">
              {student.fullName
                .split(' ')
                .map((part) => part[0])
                .slice(0, 2)
                .join('')
                .toUpperCase()}
            </div>
            <div>
              <h3 id="student-details-title" className="text-lg font-bold text-slate-900">
                {student.fullName}
              </h3>
              <span
                className={`mt-1 inline-flex items-center rounded-full border px-2.5 py-0.5 text-xs font-medium ${getDepartmentBadgeStyle(
                  student.department
                )}`}
              >
                {student.department}
              </span>
            </div>
          </div>
          <button
            type="button"
            onClick={onClose}
            aria-label="Close modal"
            className="p-2 rounded-xl text-slate-400 hover:text-slate-600 hover:bg-slate-200/60 transition-colors cursor-pointer"
          >
            <X className="w-5 h-5" />
          </button>
        </div>

        {/* Modal Body */}
        <div className="p-6 grid grid-cols-1 sm:grid-cols-2 gap-4">
          {detailItems.map((item) => {
            const IconComp = item.icon;
            return (
              <div
                key={item.label}
                className="rounded-xl border border-slate-100 bg-slate-50/60 p-3.5 flex items-start gap-3"
              >
                <div className="p-2 rounded-lg bg-white border border-slate-200/80 text-indigo-600 shrink-0">
                  <IconComp className="w-4 h-4" />
                </div>
                <div className="min-w-0">
                  <p className="text-xs font-medium text-slate-500">{item.label}</p>
                  <p className="mt-0.5 text-sm font-semibold text-slate-900 break-words">
                    {item.value}
                  </p>
                </div>
              </div>
            );
          })}
        </div>

        {/* Modal Footer */}
        <div className="px-6 py-4 bg-slate-50/70 border-t border-slate-100 flex items-center justify-end gap-2.5">
          <button
            type="button"
            onClick={() => {
              onClose();
              onEdit(student);
            }}
            className="inline-flex items-center gap-1.5 rounded-xl border border-indigo-200 bg-indigo-50 px-4 py-2 text-sm font-semibold text-indigo-700 hover:bg-indigo-100 transition-colors cursor-pointer"
          >
            <Edit2 className="w-4 h-4" />
            Edit Record
          </button>
          <button
            type="button"
            onClick={onClose}
            className="inline-flex items-center rounded-xl bg-slate-900 px-4 py-2 text-sm font-semibold text-white hover:bg-slate-800 transition-colors cursor-pointer"
          >
            Close
          </button>
        </div>
      </div>
    </div>
  );
};

// ============================================================================
// Component 6: Delete Confirmation Modal
// ============================================================================

const DeleteConfirmModal = ({ student, onCancel, onConfirm }) => {
  useEffect(() => {
    if (!student) return;
    const handleKeyDown = (e) => {
      if (e.key === 'Escape') onCancel();
    };
    window.addEventListener('keydown', handleKeyDown);
    return () => window.removeEventListener('keydown', handleKeyDown);
  }, [student, onCancel]);

  if (!student) return null;

  return (
    <div
      className="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-900/50 backdrop-blur-xs"
      role="dialog"
      aria-modal="true"
      aria-labelledby="delete-confirm-title"
      onClick={onCancel}
    >
      <div
        className="w-full max-w-md bg-white rounded-2xl shadow-xl border border-slate-200 p-6"
        onClick={(e) => e.stopPropagation()}
      >
        <div className="flex items-start gap-4">
          <div className="p-3 rounded-2xl bg-rose-50 text-rose-600 shrink-0">
            <Trash2 className="w-6 h-6" />
          </div>
          <div>
            <h3 id="delete-confirm-title" className="text-base font-bold text-slate-900">
              Delete Student Record?
            </h3>
            <p className="mt-1.5 text-sm text-slate-600">
              Are you sure you want to remove{' '}
              <span className="font-semibold text-slate-900">{student.fullName}</span> (
              <span className="font-mono text-xs">{student.studentId}</span>)? This action cannot
              be undone.
            </p>
          </div>
        </div>

        <div className="mt-6 flex items-center justify-end gap-2.5">
          <button
            type="button"
            onClick={onCancel}
            className="rounded-xl border border-slate-200 bg-white px-4 py-2 text-sm font-semibold text-slate-700 hover:bg-slate-50 transition-colors cursor-pointer"
          >
            Cancel
          </button>
          <button
            type="button"
            onClick={() => onConfirm(student)}
            className="rounded-xl bg-rose-600 px-4 py-2 text-sm font-semibold text-white hover:bg-rose-700 transition-colors cursor-pointer"
          >
            Yes, Delete
          </button>
        </div>
      </div>
    </div>
  );
};

// ============================================================================
// Component 7: Toast Notifications
// ============================================================================

const ToastContainer = ({ toasts, onDismiss }) => {
  if (toasts.length === 0) return null;

  return (
    <div
      aria-live="polite"
      className="fixed bottom-4 right-4 z-50 flex flex-col gap-2.5 max-w-sm w-[calc(100%-2rem)] sm:w-auto"
    >
      {toasts.map((toast) => {
        const isDanger = toast.type === 'delete';
        return (
          <div
            key={toast.id}
            className={`flex items-center gap-3 rounded-xl border px-4 py-3 shadow-lg bg-white text-sm font-medium transition-all ${
              isDanger
                ? 'border-rose-200 text-slate-800'
                : 'border-emerald-200 text-slate-800'
            }`}
          >
            <span
              className={`p-1.5 rounded-lg shrink-0 ${
                isDanger ? 'bg-rose-50 text-rose-600' : 'bg-emerald-50 text-emerald-600'
              }`}
            >
              {isDanger ? <Trash2 className="w-4 h-4" /> : <CheckCircle2 className="w-4 h-4" />}
            </span>
            <span className="flex-1 text-slate-700">{toast.message}</span>
            <button
              type="button"
              onClick={() => onDismiss(toast.id)}
              aria-label="Dismiss notification"
              className="p-1 rounded-lg text-slate-400 hover:text-slate-600 hover:bg-slate-100 cursor-pointer"
            >
              <X className="w-3.5 h-3.5" />
            </button>
          </div>
        );
      })}
    </div>
  );
};

// ============================================================================
// Main Application Component (Default Export)
// ============================================================================

export default function App() {
  // Load initial students from localStorage with safe fallback
  const [students, setStudents] = useState(() => {
    try {
      const saved = window.localStorage.getItem(STORAGE_KEY);
      if (saved !== null) {
        const parsed = JSON.parse(saved);
        if (Array.isArray(parsed)) return parsed;
      }
    } catch (err) {
      console.error('Failed to read students from localStorage:', err);
    }
    return INITIAL_STUDENTS;
  });

  const [editingStudent, setEditingStudent] = useState(null);
  const [viewingStudent, setViewingStudent] = useState(null);
  const [deletingStudent, setDeletingStudent] = useState(null);

  const [searchQuery, setSearchQuery] = useState('');
  const [departmentFilter, setDepartmentFilter] = useState('ALL');
  const [toasts, setToasts] = useState([]);

  const formContainerRef = useRef(null);

  // Persist student list to localStorage whenever it changes
  useEffect(() => {
    try {
      window.localStorage.setItem(STORAGE_KEY, JSON.stringify(students));
    } catch (err) {
      console.error('Failed to save students to localStorage:', err);
    }
  }, [students]);

  // Toast helper
  const showToast = (message, type = 'success') => {
    const id = `toast-${Date.now()}-${Math.random().toString(36).slice(2, 6)}`;
    setToasts((prev) => [...prev, { id, message, type }]);
    window.setTimeout(() => {
      setToasts((prev) => prev.filter((t) => t.id !== id));
    }, 3500);
  };

  const dismissToast = (id) => {
    setToasts((prev) => prev.filter((t) => t.id !== id));
  };

  // Register or Update a student
  const handleSaveStudent = (studentData, isEditMode) => {
    if (isEditMode) {
      setStudents((prev) =>
        prev.map((item) => (item.id === studentData.id ? studentData : item))
      );
      setEditingStudent(null);
      showToast(`Updated ${studentData.fullName}'s record successfully.`);
    } else {
      setStudents((prev) => [studentData, ...prev]);
      showToast(`Registered ${studentData.fullName} successfully.`);
    }
  };

  // Enter Edit Mode
  const handleEditStudent = (student) => {
    setEditingStudent(student);
    if (formContainerRef.current) {
      formContainerRef.current.scrollIntoView({ behavior: 'smooth', block: 'start' });
    }
  };

  // Cancel Edit Mode
  const handleCancelEdit = () => {
    setEditingStudent(null);
  };

  // Confirm Delete
  const handleConfirmDelete = (student) => {
    setStudents((prev) => prev.filter((item) => item.id !== student.id));
    if (editingStudent && editingStudent.id === student.id) {
      setEditingStudent(null);
    }
    setDeletingStudent(null);
    showToast(`Deleted ${student.fullName} (${student.studentId}).`, 'delete');
  };

  // Reset to sample dataset
  const handleResetDemoData = () => {
    setStudents(INITIAL_STUDENTS);
    setEditingStudent(null);
    setSearchQuery('');
    setDepartmentFilter('ALL');
    showToast('Restored default sample student records.');
  };

  // Real-time filtered students list
  const filteredStudents = useMemo(() => {
    const query = searchQuery.trim().toLowerCase();
    return students.filter((student) => {
      const matchesDepartment =
        departmentFilter === 'ALL' || student.department === departmentFilter;

      if (!matchesDepartment) return false;
      if (!query) return true;

      return (
        student.fullName.toLowerCase().includes(query) ||
        student.studentId.toLowerCase().includes(query) ||
        student.email.toLowerCase().includes(query) ||
        student.department.toLowerCase().includes(query)
      );
    });
  }, [students, searchQuery, departmentFilter]);

  const hasFilters = searchQuery.trim() !== '' || departmentFilter !== 'ALL';

  return (
    <div className="min-h-screen bg-slate-50 text-slate-900 flex flex-col">
      {/* Top Navigation / Header */}
      <header className="bg-white border-b border-slate-200/80 sticky top-0 z-30">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-4 flex items-center justify-between gap-4">
          <div className="flex items-center gap-3">
            <div className="p-2.5 rounded-xl bg-indigo-600 text-white shadow-sm">
              <GraduationCap className="w-6 h-6" />
            </div>
            <div>
              <h1 className="text-lg sm:text-xl font-bold tracking-tight text-slate-900">
                Student Registration & Management
              </h1>
              <p className="text-xs sm:text-sm text-slate-500">
                Enroll, search, update, and manage student academic records
              </p>
            </div>
          </div>

          <button
            type="button"
            onClick={handleResetDemoData}
            title="Restore the 4 sample students"
            className="inline-flex items-center gap-1.5 rounded-xl border border-slate-200 bg-white px-3 py-2 text-xs font-semibold text-slate-600 hover:bg-slate-50 hover:text-slate-900 transition-colors cursor-pointer shrink-0"
          >
            <RotateCcw className="w-3.5 h-3.5" />
            <span className="hidden sm:inline">Reset Demo Data</span>
          </button>
        </div>
      </header>

      {/* Main Content */}
      <main className="flex-1 max-w-7xl w-full mx-auto px-4 sm:px-6 lg:px-8 py-6 sm:py-8 space-y-6">
        {/* 1. Summary Stats Cards */}
        <StatsCards students={students} />

        {/* 2. Main Split Workspace: Form + Directory */}
        <div className="grid grid-cols-1 xl:grid-cols-12 gap-6 items-start">
          {/* Left Column: Registration & Edit Form */}
          <div className="xl:col-span-4 xl:sticky xl:top-24">
            <StudentForm
              students={students}
              editingStudent={editingStudent}
              onSaveStudent={handleSaveStudent}
              onCancelEdit={handleCancelEdit}
              formContainerRef={formContainerRef}
            />
          </div>

          {/* Right Column: Search/Filter + Student List */}
          <div className="xl:col-span-8 space-y-4">
            <SearchFilterBar
              searchQuery={searchQuery}
              onSearchChange={setSearchQuery}
              departmentFilter={departmentFilter}
              onDepartmentFilterChange={setDepartmentFilter}
              totalCount={students.length}
              filteredCount={filteredStudents.length}
            />

            <StudentList
              students={filteredStudents}
              editingStudentId={editingStudent?.id || null}
              onViewStudent={setViewingStudent}
              onEditStudent={handleEditStudent}
              onRequestDelete={setDeletingStudent}
              hasFilters={hasFilters}
            />
          </div>
        </div>
      </main>

      {/* Modals & Toast Notifications */}
      <StudentDetailsModal
        student={viewingStudent}
        onClose={() => setViewingStudent(null)}
        onEdit={handleEditStudent}
      />

      <DeleteConfirmModal
        student={deletingStudent}
        onCancel={() => setDeletingStudent(null)}
        onConfirm={handleConfirmDelete}
      />

      <ToastContainer toasts={toasts} onDismiss={dismissToast} />
    </div>
  );
}
